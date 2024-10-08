### 0. Azure 컴퓨팅 서비스 선택
![image](https://github.com/user-attachments/assets/a688b2cc-2747-4b22-85f5-32b24ba2a7ee)
Link: https://learn.microsoft.com/ko-kr/azure/architecture/guide/technology-choices/compute-decision-tree


### 1. Java용 신뢰할 수 있는 웹앱 패턴
- https://learn.microsoft.com/ko-kr/azure/architecture/web-apps/guides/reliable-web-app/java/apply-pattern
- 
![image](https://github.com/user-attachments/assets/c218e700-eb8b-4b2a-9b87-0c1a4e6e0042)

- 사례(Contoso Fiber)
  - AS-IS / 비즈니스 목표 설정
    - Contoso Fiber는 온-프레미스 CAMS(고객 계정 관리 시스템) 웹앱을 확장하고자 함(지역)/수요 증가 충족 필요 ==> Azure 마이그 결정
      - 저렴한 고가용성 코드 변경 적용
      - 99.9%의 SLO(서비스 수준 목표)에 도달
      - DevOps 사례 채택
      - 비용 최적화 환경 만들기
      - 안정성 및 보안 향상
  - TO-BE
    - 애플리케이션 플랫폼
      - Azure App Service 선정
        - 사유: 기존 Spring boot app 수정 최소화, 높은 SLA, 완전 관리형 호스팅, 컨테이너화 기능, auto scaling
    - ID 관리
      - Entra ID
        - 사유: 직원의 인증 및 권한 부여, 확장성, 사용자ID 컨트롤(기존 ID사용 가능), OAuth 2.0 지원
    - 데이터베이스
      - azure database for postgresql
        - 사유: 신뢰성(동일 지역 대기서버 데이터 동기 복제), 지역간 복제(읽기전용), 완전관리형으로 관리 오버헤드 감소, 마이그레이션 지원
    - app 성능 모니터링
      - Application Insight
        - 사유: 성능 이상 자동 검색, 실행중인 앱 문제 진단, 원격 분석
    - 캐시
      - Azure cache for Redis
        - 사유: 성능(잘 변하지 않는 자주 액세스하는 데이터), 모든 인스턴스에서 액세스 가능한 통합 캐시 위치, 논스틱 세션관리
    - 부하 분산 장치
      - Front Door
        - 사유: 라우팅 유연성, 트래픽 가속(anycast로 가장 빠른 경로 찾음), 사용자 지정 도메인 지원, 상태 프로브(백엔드 서버 상태 체크), 모니터링(대시보드 제공), DDoS 보호기능 제공
    - 웹 애플리케이션 방화벽
      - Azure WAF: 일반적인 웹 악용 및 취약성으로부터 중앙 집중식 보호
        - 성능저하 없이 보호 제공, 봇넷 보호, 온 프레미스와 유사한 수준의 보안정책 제공 가능
    - 비밀 관리자
      - Azure Key Vault
    - 엔드포인트 보안
      - Azure Private Link: Azure 서비스에 대한 프라이밋 전용 액세스 사용
        - Azure 서비스(Cache for Redis, DB 등)와 고객의 서비스(vnet 등) 사이에 프라이빗 액세스(공용 인터넷 미경유).
  - 아키텍처 선정

### 2. 네트워크 구성(vnet)
- Hub and spoke 아키텍처 구성
  - Hub 네트워크: 주로 공통 서비스(예: 방화벽, VPN 게이트웨이, NAT 게이트웨이 등)를 중앙에서 관리
  - Spoke 네트워크: 각기 다른 부서나 애플리케이션, 워크로드가 사용하는 개별 네트워크를 의미
  - 예시: 회사의 IT 부서가 허브 네트워크를 관리하며, 각각의 프로젝트 팀이 스포크 네트워크를 통해 해당 프로젝트에 필요한 리소스를 관리합니다. 각 프로젝트 팀은 다른 팀의 네트워크와는 격리되어 있지만, 허브 네트워크를 통해 공통 서비스에 액세스

### 3. 서킷 브레이커 패턴
- 개념:
  - 서킷 브레이커 패턴은 소프트웨어 시스템에서 장애나 실패가 발생했을 때, 전체 시스템의 기능 저하를 방지하기 위해 요청을 차단하거나 우회하는 패턴입니다. 실제 전기 회로의 서킷 브레이커와 유사하게, 특정 조건(예: 연속된 실패)에 도달하면 호출을 차단하고, 시스템이 안정될 때까지 대체 행동을 취합니다.

- 용도:
  - 적용하면 좋은 시나리오:
    - 원격 서비스 호출: 외부 API나 원격 서비스가 일시적으로 불안정할 때, 실패가 반복되는 동안 호출을 차단해 시스템 자원을 낭비하지 않도록 합니다.
    - 데이터베이스 연결: 데이터베이스가 불안정하거나 과부하 상태일 때, 요청을 차단하여 추가적인 부하를 방지합니다.
    - 마이크로서비스 아키텍처: 서비스 간의 의존성이 높은 환경에서, 특정 서비스가 실패할 경우 다른 서비스로 장애가 전파되지 않도록 보호합니다.
  - 적용하면 안 되는 시나리오:
    - 중요한 요청이나 트랜잭션: 서킷 브레이커가 활성화되면 중요한 요청이 차단될 수 있어, 비즈니스 크리티컬한 트랜잭션에는 적합하지 않을 수 있습니다.
    - 단일 장애 지점: 서킷 브레이커 패턴이 없는 경우, 한 번의 장애로 전체 시스템이 멈출 수 있는 구조에서는 적용이 적합하지 않습니다.
    - 응답 시간이 중요한 작업: 서킷 브레이커의 상태 확인 및 전환 과정에서의 추가적인 지연으로 인해, 응답 시간이 중요한 작업에선 부적합할 수 있습니다.

### 4. 다시 시도 패턴
- 일시적인 오류에 대한 대응(일반적으로 몇 초 내에 해결)
- Java에서는 Resilience4j 이용 ==> 서킷 브레이커, 재시도, 속도 제한기 등 패턴 제공

### 5. Azure Cache for Redis를 이용한 세션 관리 장점
- 논스틱 세션: Azure Cache for Redis를 사용하면 세션 데이터를 웹앱 서버 메모리에 저장하지 않고, Redis 캐시 서버에 저장할 수 있습니다. 이렇게 하면 웹앱 서버들이 특정 세션 데이터를 항상 같은 서버에 저장하지 않아도 됩니다. 즉, "스틱 세션"이 아닌 "논스틱 세션"이 됩니다. 이는 서버 간에 세션 데이터를 공유할 수 있어, 어느 서버에서든지 동일한 세션 데이터에 접근할 수 있음을 의미합니다.
- 확장성: 서버의 메모리에 세션 데이터를 저장하지 않기 때문에, 서버의 메모리 사용량이 증가하지 않습니다. 서버를 확장(더 많은 서버를 추가)할 때도, 세션 데이터가 Redis 캐시에 있으므로 추가된 서버들이 바로 동일한 세션 데이터를 사용할 수 있어 확장성이 좋습니다.
- 성능 개선: Redis는 매우 빠르고 효율적인 캐시 서비스이므로, 세션 데이터를 Redis에 저장하면 접근 속도가 빨라지고 성능이 향상됩니다.
- 관리 용이성: Redis는 완전히 관리되는 서비스이므로, 사용자가 직접 서버를 관리할 필요 없이, 쉽게 세션 관리를 설정하고 운영할 수 있습니다.

이 모든 장점 덕분에, 웹앱이 더 확장 가능하고, 성능이 좋으며, 관리하기 쉬운 상태로 세션 관리를 할 수 있게 됩니다.

### 6. 동시성 해소를 위한 큐잉 시스템 솔루션
동시성 이슈를 해결하기 위해 큐잉 시스템을 사용하는 것은 매우 효과적인 방법입니다. 이를 위해 적합한 솔루션을 몇 가지 제안드리겠습니다.

1. Azure Service Bus
  - 설명: Azure Service Bus는 클라우드 기반의 메시징 서비스로, 복잡한 비동기 메시지 통신을 지원합니다. 서비스 버스를 사용하면 메시지를 순차적으로 처리하고, 동시성 문제를 방지할 수 있습니다.
  - 장점:
    - 순서 보장: 메시지가 도착한 순서대로 처리되도록 FIFO(First In, First Out) 큐를 지원합니다.
    - 트랜잭션 지원: 여러 메시지를 하나의 트랜잭션으로 처리할 수 있어, 메시지가 안전하게 처리되도록 보장합니다.
    - 큐와 토픽: 단순한 큐잉뿐만 아니라 Pub/Sub 모델을 지원하여 다양한 시나리오에 대응할 수 있습니다.
2. Azure Queue Storage
  - 설명: Azure Queue Storage는 간단한 메시지 큐잉을 제공하는 Azure 서비스로, 비동기적 메시지 처리가 필요할 때 사용됩니다.
  - 장점:
    - 간단한 사용: 기본적인 큐잉 기능을 제공하며, 손쉽게 구현할 수 있습니다.
    - 저렴한 비용: 매우 저렴한 비용으로 대규모 메시지를 처리할 수 있습니다.
    - 지속성: 메시지는 클라우드에 안전하게 저장되어 손실의 위험이 없습니다.
3. RabbitMQ (Azure VM 또는 Azure Kubernetes Service에서 호스팅)
  - 설명: RabbitMQ는 오픈 소스 메시지 브로커로, Azure VM이나 Azure Kubernetes Service(AKS)에서 호스팅할 수 있습니다. 높은 수준의 메시지 큐잉 기능을 제공하며, 다양한 프로토콜을 지원합니다.
  - 장점:
    - 다양한 프로토콜 지원: AMQP, MQTT 등 다양한 프로토콜을 지원하여 유연한 아키텍처를 구축할 수 있습니다.
    - 확장성: 클러스터링과 페더레이션을 통해 쉽게 확장할 수 있어, 대규모 시스템에서도 사용이 가능합니다.
    - 풍부한 기능: 메시지 우선순위, 딜레이드 메시징, 재시도 등 고급 기능을 제공합니다.
4. Azure Functions with Durable Functions
  - 설명: Azure Functions는 서버리스 컴퓨팅을 제공하는 Azure 서비스이며, Durable Functions를 사용하면 상태를 관리하면서 워크플로우를 설계할 수 있습니다.
  - 장점:
    - 상태 관리: Durable Functions를 사용하면 비동기 프로세스를 쉽게 관리할 수 있습니다.
    - 내장된 큐잉 기능: Azure Functions 자체가 트리거 기반으로 작동하며, 메시지 큐잉과 비슷한 동작을 내장하고 있어, 메시지 처리 순서를 관리할 수 있습니다.
    - 서버리스 비용 모델: 서버리스 모델로 실행되기 때문에 필요한 경우에만 비용이 발생하여 효율적입니다.
5. Kafka (Azure HDInsight or Confluent Cloud on Azure)
  - 설명: Apache Kafka는 분산 스트리밍 플랫폼으로, 대규모 실시간 데이터를 처리할 수 있습니다. Azure HDInsight에서 Kafka를 제공하거나, Confluent Cloud의 Kafka 서비스를 사용할 수 있습니다.
  - 장점:
    - 고속 데이터 처리: 실시간으로 대규모 데이터를 빠르게 처리할 수 있습니다.
    - 확장성: 분산 아키텍처를 통해 시스템을 손쉽게 확장할 수 있습니다.
    - 실시간 스트리밍: 데이터 스트리밍과 로그 수집 등의 다양한 실시간 애플리케이션에 적합합니다.

**결론:**
간단한 비동기 처리가 필요한 경우에는 Azure Queue Storage나 Azure Service Bus를 사용하는 것이 좋습니다.
복잡한 시나리오나 고성능이 요구되는 경우에는 RabbitMQ나 Kafka를 고려해볼 수 있습니다.
서버리스 환경에서는 Azure Functions with Durable Functions가 적합합니다.
이들 솔루션을 사용하면 동시성 이슈를 효과적으로 해결하면서도 시스템의 확장성과 안정성을 높일 수 있습니다.

### 7. 마이크로서비스 아키텍처
마이크로서비스 아키텍처의 모범 사례를 보여드리기 위해 드론 배달 애플리케이션이라는 참조 구현을 만들었습니다. 이 구현은 AKS(Azure Kubernetes Service)를 사용하여 Kubernetes에서 실행됩니다. 참조 구현은 GitHub에서 찾을 수 있습니다.
![image](https://github.com/user-attachments/assets/ca072bcc-c3ef-42cb-bf53-56da6baf1f8b)

1. 마이크로 서비스를 위한 Azure 컴퓨팅 옵션 선택(https://learn.microsoft.com/ko-kr/azure/architecture/microservices/design/compute-options)
  - 서비스 오케스트레이터: AKS, Azure Container Apps, Service Fabric, Docker Enterprise Edition(IaaS)
  - 컨테이너
  - 서버리스(Function as a Service)

2. 서비스 간 통신

3. API 디자인

4. API 게이트웨이

5. 데이터 고려사항

6. 컨테이너 오케스트레이션

7. 마이크로서비스 디자인 패턴(https://learn.microsoft.com/ko-kr/azure/architecture/microservices/design/patterns)
![image](https://github.com/user-attachments/assets/30e0c00a-95c0-4c83-923c-37e7ee09c1d2)

8. 마이크로서비스 아키텍처로 마이그레이션
  - DDD를 이용해 모놀리식 App을 마이크로서비스로 마이그레이션(https://learn.microsoft.com/ko-kr/azure/architecture/microservices/migrate-monolith)
9. ...
