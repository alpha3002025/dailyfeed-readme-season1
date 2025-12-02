
dailyfeed-installer 프로젝트 내에는 dailyfeed-infrastructure, dailyfeed-app-helm 모듈이 있습니다. 각각은 초기 개발 시에는 모두 local 프로필에 대해서는 수작업으로 하나씩 쉘스크립트를 작성하고 실행 구조를 정의했고, local 프로필에 대한 스크립트가 한벌이 만들어졌을때 이 local 프로필 기반의 스크립트들을 기반으로 dev 프로필 기반의 스크립트 들을 AI 에이전트(Claude Code)를 이용해서 dev 프로필 기반의 스크립트를 만들었습니다.<br/>

처음부터 대부분의 내용을 AI 를 이용해서 인프라 스크립트를 작성하면 AI 와 결국엔 싸우게 될것 같아서... 처음에는 원래 알고 있던 지식을 기반으로 구조를 잡으면서 디버깅을 할때에만 AI의 도움을 받았고, 완성된 한 벌이 만들어졌을 때 dev 기반의 스크립트, local-hybrid 기반의 스크립트를 구성했습니다.<br/>
<br/>

```
dailyfeed-installer
  ㄴ dailyfeed-infrastructure
  ㄴ dailyfeed-app-helm
```
<br/>
<br/>

# `dailyfeed-infrastructure`
> Project Github : https://github.com/alpha3002025/dailyfeed-infrastructure

<br/>

infrastructure 를 설치하는 과정에 대한 프로젝트입니다. `dailyfeed-infrastructure/install-local-hybrid.sh` 를 통해 infra 들이 설치되며 다음의 작업들을 수행합니다.<br/>
- `Kind` 기반의 k8s 클러스터 설치
- `MySQL, MongoDB, Kafka, Redis` 를 docker-compose 기반으로 설치
- `Kind` 의 control plane 컨테이너, worker node 컨테이너 들을 docker-compose 네트워크에 connect
<br/>

**`install-local.sh` -`local` 설치 스크립트**<br/>
개발 초반에는 `install-local.sh` 에 설치 스크립트를 작성했고, 이 스크립트 내에서는 MySQL, MongoDB, Kafka, Redis 를 모두 helm 으로 함께 kubernetes 클러스터 내에 함께 설치하도록 되어 있습니다. 하지만 이 방식은 kubernetes 내에 MySQL, MongoDB, Kafka, Redis 를 helm 으로 배포했기에 서비스 애플리케이션들과 클러스터의 리소스를 공유하게 되기에 애플리케이션의 정확한 리소스 사용량을 파악하기 쉽자 않다는 단점이 있었습니다. 또한 HPA 설정 시 인프라(MySQL, MongoDB, Kafka, Redis)와 서비스 애플리케이션 간 리소스 경합이 발생하기에 HPA 설정시 정확한 리소스 사용량 부여에 어려움을 겪게되었습니다.<br/>

즉, 인프라(MySQL, MongoDB, Kafka, Redis) 리소스와 서비스 애플리케이션 리소스 간 분리가 이뤄지지 않아서 서로에게 영향을 주는 현상이 있었습니다. 이런 이유로 아래에서 설명할 `install-local-hybrid.sh` 스크립트로의 전환 작업을 시작하게 됐습니다.<br/>
<br/>

**`install-local-hybrid.sh` - `local` ➝ `local-hybrid`**<br/>
이 방식은 인프라(MySQL, MongoDB, Kafka, Redis)는 `docker-compose.yaml` 기반으로 작성합니다. 그리고 서비스 애플리케이션 들은 helm 을 통해 배포됩니다. 인프라(MySQL, MongoDB, Kafka, Redis) 영역은 별도의 영역에서 실행되며, HPA 로 클러스터 내에서 관리되어야 할 서비스 애플리케이션들온 helm 을 통해 쿠버네티스에서 실행되도록 했습니다.<br/>

- 인프라(MySQL, MongoDB, Kafka, Redis) : Docker Compose 기반으로 설치, 독립된 영역에 설치 (dailyfeed-infrastructure 에 정의)
- 서비스 애플리케이션 : helm 기반으로 설치, kind 기반 kubernetes 내에 설치 (dailyfeed-app-helm 내에 정의)

이때 서로 분리된 영역에서 인프라와 서비스 애플리케이션이 동작하기에 서로의 docker network 를 연결시켜줘야 하는데, 이 작업은 kind 의 control plane node, worker node 의 각 컨테이너에 docker connect 를 통해 각 network 를 연결해주는 script 를 추가해서 해결했으며 관련된 소스코드의 일부는 다음과 같습니다.<br/>
<br/>


**`dailyfeed-infrastructure/install-local-hybrid.sh`**<br/>
```sh

### ...


# Kind 클러스터의 컨테이너를 dailyfeed-network에 연결
NETWORK_NAME="local-hybrid_dailyfeed-network"

# Kind 컨트롤 플레인 노드 연결
KIND_CONTROL_PLANE="istio-cluster-control-plane"
if docker ps --format '{{.Names}}' | grep -q "^${KIND_CONTROL_PLANE}$"; then
    echo "  → Connecting ${KIND_CONTROL_PLANE} to ${NETWORK_NAME}..."
    docker network connect ${NETWORK_NAME} ${KIND_CONTROL_PLANE} 2>/dev/null || echo "  ✓ Already connected"
else
    echo "  ⚠️  ${KIND_CONTROL_PLANE} not found"
fi

# Kind 워커 노드들 연결 (있는 경우)
for worker in $(docker ps --format '{{.Names}}' | grep "^istio-cluster-worker"); do
    echo "  → Connecting ${worker} to ${NETWORK_NAME}..."
    docker network connect ${NETWORK_NAME} ${worker} 2>/dev/null || echo "  ✓ Already connected"
done

echo "  ✅ Network connection completed"
echo ""

echo "=== 🔗 Connecting Docker Compose infrastructure to Kind network ==="
echo "This allows bidirectional communication between Docker Compose and Kubernetes"

# Docker Compose 인프라 컨테이너들을 Kind 네트워크에 연결
KIND_NETWORK="kind"

# Kafka 브로커들 연결
for kafka in kafka-1 kafka-2 kafka-3; do
    if docker ps --format '{{.Names}}' | grep -q "^${kafka}$"; then
        echo "  → Connecting ${kafka} to ${KIND_NETWORK}..."
        docker network connect ${KIND_NETWORK} ${kafka} 2>/dev/null || echo "  ✓ Already connected"
    else
        echo "  ⚠️  ${kafka} not found"
    fi
done

# MongoDB 레플리카셋 연결
for mongo in mongo-dailyfeed-1 mongo-dailyfeed-2 mongo-dailyfeed-3; do
    if docker ps --format '{{.Names}}' | grep -q "^${mongo}$"; then
        echo "  → Connecting ${mongo} to ${KIND_NETWORK}..."
        docker network connect ${KIND_NETWORK} ${mongo} 2>/dev/null || echo "  ✓ Already connected"
    else
        echo "  ⚠️  ${mongo} not found"
    fi
done

# Redis 연결
if docker ps --format '{{.Names}}' | grep -q "^redis-dailyfeed$"; then
    echo "  → Connecting redis-dailyfeed to ${KIND_NETWORK}..."
    docker network connect ${KIND_NETWORK} redis-dailyfeed 2>/dev/null || echo "  ✓ Already connected"
else
    echo "  ⚠️  redis-dailyfeed not found"
fi

# MySQL 연결
if docker ps --format '{{.Names}}' | grep -q "^mysql-dailyfeed$"; then
    echo "  → Connecting mysql-dailyfeed to ${KIND_NETWORK}..."
    docker network connect ${KIND_NETWORK} mysql-dailyfeed 2>/dev/null || echo "  ✓ Already connected"
else
    echo "  ⚠️  mysql-dailyfeed not found"
fi

echo "  ✅ Infrastructure network connection completed"
```
<br/>

**`install-dev.sh`**<br/>
MySQL, MongoDB 를 외부 클라우드 업체에서 제공하는 것을 사용하는 버전의 설치 스크립트입니다. `install-local.sh`를 만들때 들었던 시간과 정성을 `install-dev.sh`에도 똑같이 쏟아부을 수 여건이 되지 않아서 시간관계상 AI 를 활용해야 했고, `install-dev.sh` 의 경우 Claude Code 내에서 `install-local.sh` 를 참고해서 `install-dev.sh`에서 사용할 dev 환경의 RDS, Atlas Mongodb 의 주소 세팅, 접속 계정 등을 Secret,Service 등을 통해 지정햐도록 했습니다.<br/>

Claude Code 도 한번에 정확한 답을 내지는 않기에 자주 확인 후 증상을 수정해나가는 과정을 거쳤으며, 서비스 애플리케이션 측 코드를 local 환경에서 개발 작업을 하면서 주기적으로 Claude Code 로 작업이 진행된 내용을 확인하면서 인프라 쪽의 작업도 확인해나가는 과정을 거쳤습니다.<br/>

dev 환경의 경우 `install-dev.sh`는 아직 완전하게 마음에 들지는 않습니다. 접속 계정, 접속 주소 등을 secret 에 BASE64 인코딩을 해서 평문으로만 들어가지 않도록 해두어 github repository 내의 인프라 주소가 평문 검색으로 조회되지 않도록 해두었다는 것 만으로는 아직까지는 만족감을 느끼지는 않지만, 시간 관계상 이 정도 까지만 완성을 해두기로 결정했습니다.<br/>
<br/>

현재 dev 환경의 infra 는 로컬에서만 접속이 됩니다. 클라우드 인프라 내에 dev 프로필로 애플리케이션들을 올려두기에는 비용상으로 압박이 좀 있기에, 로컬에서 dev 프로필로 접속이 되도록 해두었고, 클라우드 인프라 내에 모든 애플리케이션을 배포했을 때 이론상으로는 모두 이상 없이 동작한다고 볼수는 있지만, 실제 dev 환경을 구성하지는 않았습니다.<br/>
<br/>

season 2 로 개발하려는 새로운 버전의 프로젝트에서는 로컬에서 dev 인프라에 접속할때에 대해 접속주소 등을 가지고 있는 docker image 기반의 소형 애플리케이션을 미리 만들어서, 이 애플리케이션을 sidecar 로 두어서 배포하거나, Config Server를 운영하거나 하는 등의 전략들을 생각 중 입니다.<br/>
<br/>


# dailyfeed-app-helm
> Project Github : https://github.com/alpha3002025/dailyfeed-app-helm


WIP




