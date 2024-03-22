# 서론

단일체 애플리케이션을 배포한다는 것은 대규모 애플리케이션의 여러 복사본을 실행하는 것을 의미하며, 일반적으로 여러 대의 서버(물리적 또는 가상)를 제공하고 여러 개의 애플리케이션 인스턴스를 실행합니다. 단일체 애플리케이션을 배포하는 것은 간단하지 않지만, 마이크로서비스 애플리케이션을 배포하는 것보다는 확실히 더 단순합니다.

마이크로서비스 애플리케이션은 수백 개의 서비스로 구성되어 있으며, 이 서비스들은 다양한 언어와 프레임워크로 개발될 수 있습니다. 각 서비스는 독립된 애플리케이션이며, 자체적인 배포, 리소스, 확장 및 모니터링 요구사항을 가질 수 있습니다. 예를 들어, 서비스의 요구에 따라 여러 서비스 인스턴스를 실행할 수 있으며, 각 인스턴스는 자체 CPU, 메모리 및 I/O 자원을 필요로 합니다. 복잡함에도 불구하고 더 큰 도전은 서비스 배포가 빠르고, 신뢰할 수 있으며, 비용 효율적이어야 한다는 것입니다.

여러 가지 마이크로서비스 배포 패턴이 있으며, 먼저 각 호스트에서 여러 서비스 인스턴스를 실행하는 패턴부터 논의해보겠습니다.

# 단일 호스트 다중 서비스 인스턴스 패턴

마이크로서비스를 배포하는 한 가지 방법은 단일 호스트 다중 서비스 인스턴스 패턴을 사용하는 것입니다. 이 패턴을 사용하려면, 여러 대의 물리적 또는 가상 머신을 준비하고, 각 머신에서 여러 서비스 인스턴스를 실행해야 합니다. 많은 경우, 이것은 전통적인 애플리케이션 배포 방법입니다. 각 서비스 인스턴스는 하나 이상의 호스트의 잘 알려진 포트에서 실행되며, 호스트는 애완동물처럼 간주될 수 있습니다.

아래 그림은 이러한 구조를 보여줍니다:

![deployment-strategy-1](./images/deployment-strategy-1.png)

이 패턴에는 몇 가지 변수가 있습니다. 하나의 변수는 각 서비스 인스턴스가 몇 개의 프로세스로 구성되어 있는지를 나타냅니다. 예를 들어, Apache Tomcat Server에서 웹 애플리케이션으로 Java 서비스 인스턴스를 배포해야 할 수 있습니다. Node.js 서비스 인스턴스는 하나의 부모 프로세스와 여러 개의 자식 프로세스로 구성될 수 있습니다.

또 다른 변수는 동일한 프로세스 그룹 내에서 몇 개의 서비스 인스턴스가 실행되는지를 정의합니다. 예를 들어, 동일한 Apache Tomcat Server에서 여러 Java 웹 애플리케이션을 실행하거나, 동일한 OSGi 컨테이너 내에서 여러 OSGi 번들 인스턴스를 실행할 수 있습니다.

단일 호스트 다중 서비스 인스턴스 패턴은 장단점이 있습니다. 주요 장점 중 하나는 리소스 활용의 효율성입니다. 여러 서비스 인스턴스가 서버와 운영 체제를 공유함으로써, 프로세스 그룹이 여러 서비스 인스턴스를 실행할 때 효율성이 더 높아질 수 있습니다. 예를 들어, 여러 웹 애플리케이션이 동일한 Apache Tomcat Server와 JVM을 공유합니다.

또 다른 장점은 서비스 인스턴스의 배포가 빠르다는 것입니다. 서비스를 호스트에 복사하고 시작하기만 하면 됩니다. 서비스가 Java로 작성된 경우, JAR 또는 WAR 파일만 복사하면 됩니다. Node.js나 Ruby와 같은 다른 언어의 경우, 소스 코드를 복사해야 합니다. 즉, 네트워크 부하가 매우 낮습니다.

서비스 시작도 빠릅니다. 서비스가 자체적인 프로세스로 존재한다면, 단순히 시작하기만 하면 됩니다. 반면, 컨테이너 프로세스 그룹 내의 특정 서비스 인스턴스인 경우, 컨테이너에 동적으로 배포하거나 컨테이너를 재시작해야 할 수 있습니다.

단일 호스트 다중 서비스 인스턴스 접근법의 장점 외에도 여러 중대한 단점이 있습니다. 하나의 주요 단점은 서비스 인스턴스 간에 거의 또는 전혀 격리가 이루어지지 않는다는 것입니다. 각 서비스 인스턴스가 독립된 프로세스로 운영되지 않는 한, 개별 인스턴스의 리소스 사용을 정확하게 모니터링하고 제한하는 것이 어렵습니다. 따라서 하나의 서비스 인스턴스가 호스트의 모든 메모리나 CPU를 사용할 수 있는 상황이 발생할 수 있습니다.

동일한 프로세스 내에서 여러 서비스 인스턴스 간에 격리가 없을 때, 모든 인스턴스가 예를 들어 같은 JVM 힙을 공유할 수 있습니다. 이는 나쁜 서비스 인스턴스가 같은 프로세스 내의 다른 서비스를 쉽게 공격할 수 있음을 의미하며, 각 서비스 인스턴스가 사용하는 리소스의 양을 모니터링하는 것이 불가능할 수도 있습니다.

또 다른 심각한 문제는 운영 팀이 서비스를 배포하는 방법의 세부 사항을 정확히 알아야 한다는 것입니다. 서비스가 다양한 언어와 프레임워크로 작성될 수 있기 때문에, 개발 팀은 운영 팀과 많은 사항을 소통해야 합니다. 이로 인해 복잡성이 증가하고 배포 과정에서 오류가 발생할 가능성이 높아집니다.

이러한 문제에도 불구하고 단일 호스트 다중 서비스 인스턴스 접근법이 익숙할 수 있지만, 이 접근법은 여러 중대한 단점을 가지고 있습니다. 다음으로, 이러한 문제를 피할 수 있는 다른 마이크로서비스 배포 방법을 살펴보겠습니다.


# 단일 호스트 단일 서비스 인스턴스 모드 

마이크로서비스를 배포하는 또 다른 방법은 단일 호스트 단일 인스턴스 모드입니다. 이 모델을 사용할 때 호스트의 각 서비스 인스턴스는 독립적입니다. 단일 가상 머신 단일 인스턴스와 단일 컨테이너 단일 인스턴스라는 두 가지 구현 모드가 있습니다. 

## 단일 가상 머신 단일 인스턴스 모드 

그러나 단일 가상 머신 단일 인스턴스 모드에서는 일반적으로 서비스가 Amazon EC2 AMI와 같은 가상 머신 이미지(이미지)로 패키징됩니다. 각 서비스 인스턴스는 이 이미지를 사용하여 시작된 VM(예: EC2 인스턴스)입니다. 다음 그림은 이 아키텍처를 보여줍니다. 

![deployment-strategy-2](./images/deployment-strategy-2.png) 

Netfix는 이 아키텍처를 사용하여 비디오 스트리밍 서비스를 배포합니다. Netfix는 Aminator를 사용하여 각 서비스를 EC2 AMI로 패키징합니다. 실행 중인 각 서비스 인스턴스는 EC2 인스턴스입니다. 

자신만의 VM을 구축하는 데 사용할 수 있는 도구가 많이 있습니다. Aminator가 서비스를 EC2 AMI에 패키징하지 못하도록 CI(지속적 통합) 서비스(예: Jenkins)를 구성할 수 있습니다. packer.io는 자동화된 가상 머신 이미지 생성을 위한 또 다른 옵션입니다. Aminator와 달리 EC2, DigitalOcean, VirtualBox 및 VMware와 같은 다양한 가상화 기술을 지원합니다. Boxfuse는

다음과 같은 단점을 극복하는 가상 머신 이미지 생성에 대한 혁신적인 접근 방식을 갖추고 있습니다. Boxfuse는 Java 애플리케이션을 최소한의 가상 머신 이미지로 패키징하여 빠르게 생성되고 빠르게 시작되며 외부에 노출되는 서비스 인터페이스가 적기 때문에 더욱 안전합니다. 

CloudNative 회사에는 EC2 AMI 생성을 위한 SaaS 애플리케이션인 Bakery가 있습니다. 사용자의 마이크로서비스 아키텍처가 테스트를 통과한 후에는 자체 CI 서버를 구성하여 Bakery를 활성화할 수 있습니다. 베이커리는 서비스를 AMI로 패키지합니다. Bakery와 같은 SaaS 애플리케이션을 사용하면 사용자가 자체 AMI 생성 아키텍처를 설정하는 데 시간을 낭비할 필요가 없습니다. 

VM별 서비스 인스턴스 모델에는 많은 장점이 있습니다. 주요 VM 장점은 각 서비스 인스턴스가 자체 독립 CPU 및 메모리를 사용하여 완전히 독립적으로 실행되며 다른 서비스가 점유하지 않는다는 것입니다. 

또 다른 이점은 사용자가 AWS에서 제공하는 것과 같은 성숙한 클라우드 아키텍처를 사용할 수 있고 클라우드 서비스가 로드 밸런싱 및 확장성과 같은 유용한 기능을 제공한다는 것입니다. 

또 다른 이점은 서비스 구현 기술이 자립적이라는 것입니다. 서비스가 VM에 패키지되면 블랙박스가 됩니다. VM 관리 API는 배포 서비스 API가 되어 배포가 매우 간단하고 안정적이 됩니다. 

단일 가상 머신 단일 인스턴스 모델에도 단점이 있습니다. 한 가지 단점은 자원 활용이 효율적이지 않다는 것입니다. 각 서비스 인스턴스는 운영 체제를 포함하여 전체 가상 머신의 리소스를 차지합니다. 게다가 일반적인 퍼블릭 IaaS 환경에서는 가상 머신 리소스가 표준화되어 있어 완전히 활용되지 않을 수도 있습니다. 

게다가 퍼블릭 IaaS는 가상머신의 사용 여부와 상관없이 VM을 기준으로 요금이 부과됩니다. 예를 들어 AWS는 자동 확장 기능을 제공하지만 온디맨드 애플리케이션에 대한 빠른 응답이 부족하여 사용자가 더 많은 가상 머신을 배포해야 하므로 배포 비용이 증가합니다. .

또 다른 단점은 새 버전의 서비스 배포가 느리다는 것입니다. 가상 머신 이미지는 크기 때문에 생성 속도가 느리며, 같은 이유로 가상 머신 초기화도 느리고 운영 체제 시작에도 시간이 걸립니다. 그러나 항상 그런 것은 아니며 Boxfuse를 사용하여 생성된 것과 같은 일부 경량 가상 머신이 더 빠릅니다. 

세 번째 단점은 운영팀이 많은 사용자 정의를 담당한다는 것입니다. 가상 머신을 생성하고 관리하는 많은 작업을 덜어줄 수 있는 Boxfuse와 같은 도구를 사용하지 않는 한, 그렇지 않으면 핵심 비즈니스와 밀접하게 관련되지 않은 작업을 수행하는 데 많은 시간을 소비하게 됩니다. 

그럼 가상머신 특성은 그대로 유지하면서 상대적으로 가벼운 또 다른 마이크로서비스 배포 방법을 살펴보겠습니다. 

# 단일 컨테이너 단일 서비스 인스턴스 모드

모드를 사용하면 각 서비스 인스턴스가 자체 컨테이너에서 실행됩니다. 컨테이너는 운영 체제 수준에서 실행되는 가상화 메커니즘입니다. 컨테이너에는 샌드박스에서 실행되는 여러 프로세스가 포함되어 있습니다. 프로세스 관점에서 보면 자체 네임스페이스와 루트 파일 시스템이 있으므로 컨테이너의 메모리와 CPU 리소스가 제한될 수 있습니다. 일부 컨테이너에는 I/O 제한도 있으며 이러한 컨테이너 기술에는 Docker 및 Solaris 영역이 포함됩니다. 

다음 그림은 이 패턴을 보여줍니다. 

![deployment-strategy-3](./images/deployment-strategy-3.png) 

이 패턴을 사용하려면 서비스를 컨테이너 이미지로 패키징해야 합니다. 컨테이너 이미지는 서비스를 실행하는 데 필요한 라이브러리와 애플리케이션을 포함하는 파일 시스템입니다. 일부 컨테이너 이미지는 완전한 Linux 루트 파일 시스템으로 구성되어 있고 다른 이미지는 경량입니다. 예를 들어 Java 서비스를 배포하려면 Java 런타임, Apache Tomcat 서버 및 컴파일된 Java 애플리케이션을 포함하는 컨테이너 이미지를 생성해야 합니다. 

서비스가 컨테이너 이미지로 패키징되면 여러 컨테이너를 시작해야 합니다. 일반적으로 물리적 또는 가상 머신에서 여러 컨테이너를 실행하려면 컨테이너를 관리하기 위해 k8s 또는 Marathon과 같은 클러스터 관리 시스템이 필요할 수 있습니다. 클러스터 관리 시스템은 호스트를 리소스 풀로 사용하고 각 컨테이너의 리소스 요구 사항에 따라 컨테이너를 예약할 호스트를 결정합니다. 

단일 컨테이너 단일 서비스 인스턴스 모델에도 장점과 단점이 있습니다. 컨테이너의 장점은 가상 머신과 매우 유사하며 서비스 인스턴스는 완전히 독립적이며 각 컨테이너가 소비하는 리소스를 쉽게 모니터링할 수 있습니다. 가상 머신과 마찬가지로 컨테이너는 격리 기술을 사용하여 서비스를 배포합니다. 컨테이너 관리 API는 관리 서비스용 API로도 사용할 수 있습니다. 

그러나 가상 머신과 달리 컨테이너는 경량 기술입니다. 컨테이너 이미지는 빠르게 생성됩니다. 예를 들어 Spring Boot 애플리케이션을 컨테이너 이미지로 패키징하는 데는 노트북에서 단 5초밖에 걸리지 않습니다. 운영 체제 시작 메커니즘이 필요하지 않으므로 컨테이너가 빠르게 시작됩니다. 컨테이너가 시작되면 백그라운드 서비스가 시작됩니다. 

컨테이너를 사용하면 몇 가지 단점도 있습니다. 컨테이너 아키텍처가 빠르게 발전하고 있지만 아직 가상 머신 아키텍처만큼 성숙하지는 않습니다. 그리고 호스트 OS 커널은 컨테이너 간에 공유되기 때문에 가상 머신만큼 안전하지 않습니다. 

또한 컨테이너 기술은 컨테이너 이미지 관리를 위한 많은 맞춤형 요구 사항을 제시할 것이며 Google Container Engine 또는 Amazon EC2 Container Service(ECS)를 사용하지 않는 한 사용자는 컨테이너 아키텍처와 가상 머신 아키텍처를 모두 관리해야 합니다. 

셋째, 컨테이너는 가상 머신을 기반으로 요금이 부과되는 아키텍처에 배포되는 경우가 많으며, 당연히 고객은 부하 증가에 대처하기 위해 배포 비용도 증가하게 됩니다. 

흥미롭게도 컨테이너와 가상 머신의 구분이 점점 모호해지고 있습니다. 앞서 언급했듯이 Boxfuse 가상 머신은 빠르게 시작되고 생성되며 Clear Container 기술은 경량 가상 머신을 생성하도록 설계되었습니다. 유니커널 기업의 기술도 주목을 받고 있는데 최근 도커가 유니커널 기업을 인수했다. 

이 외에도 앞서 언급한 컨테이너 및 VM 기술의 단점을 보완한 서버리스 배포 기술이 주목을 받고 있다. 아래를 살펴보겠습니다. 

# 서버리스 배포 

AWS Lambda는 Java, Node.js 및 Python 서비스를 지원하는 서버리스 배포 기술의 한 예이며, 배포하려면 서비스를 ZIP 파일로 패키징하고 AWS Lambda에 업로드해야 합니다. 서비스 요청(이벤트)을 처리하는 함수의 이름을 제공하는 메타데이터가 제공될 수 있습니다. AWS Lambda는 충분한 요청을 처리하는 마이크로서비스를 자동으로 실행하지만 실행 시간과 소비된 메모리 양을 기준으로만 요금이 청구됩니다. 물론 악마는 세부 사항에 있으며 AWS Lambda에도 한계가 있습니다. 그러나 누구도 서버, 가상 머신, 컨테이너의 매우 흥미로운 측면에 대해 걱정할 필요가 없습니다. 

Lambda 함수는 상태 비저장 서비스입니다. 요청은 일반적으로 AWS 서비스를 활성화하여 처리됩니다. 예를 들어, 이미지가 S3 버킷에 업로드되고 Lambda 함수가 활성화되면 DynamoDB 이미지 테이블에 항목을 삽입하고 Kinesis 스트림에 메시지를 게시하고 이미지 처리 작업을 트리거할 수 있습니다. Lambda 기능은 타사 웹 서비스를 통해서도 활성화할 수 있습니다. 

Lambda 기능을 활성화하는 방법에는 4가지가 있습니다. 

- 웹 서비스 요청을 사용하는 직접 방법 
- AWS S3, DynamoDB, Kinesis 또는 Simple Email Service에서 생성된 이벤트에 응답하는 자동 방법 
- AWS API를 통해 애플리케이션 클라이언트 문제를 처리하는 자동 방법 게이트웨이 HTTP 요청 
- 타이밍 방법, cron을 통한 응답 - 타이머 방법과 매우 유사 

AWS Lambda는 마이크로서비스를 배포하는 매우 편리한 방법임을 알 수 있습니다. 요청 기반 과금 방식은 사용자가 자신의 비즈니스 처리 부하만 부담하면 되며, 인프라에 대한 이해가 필요하지 않기 때문에 사용자는 자신의 애플리케이션만 개발하면 됩니다. 

그러나 여전히 많은 한계가 있습니다. 타사 프록시에서 전달된 메시지를 사용하는 등 장기 서비스를 배포하는 데 사용할 필요가 없습니다. 요청은 300초 이내에 완료되어야 합니다. 이론적으로 AWS Lambda는 독립적인 인스턴스를 생성하므로 서비스는 상태 비저장이어야 합니다. ;지원되는 언어로 이루어져야 하며, 서비스가 빠르게 시작되어야 하며, 그렇지 않으면 시간 초과로 인해 중지됩니다. 
마이크로서비스 애플리케이션을 배포하는 것도 어려울 수 있습니다. 다양한 언어와 프레임워크로 작성된 수백 가지 서비스가 있습니다. 각 서비스는 고유한 배포, 리소스, 확장 및 모니터링 요구 사항을 갖춘 미니 애플리케이션입니다. 단일 가상 머신과 단일 인스턴스, 단일 컨테이너와 단일 인스턴스를 포함한 여러 마이크로서비스 배포 모델이 있습니다. 또 다른 선택 모드는 서버리스 접근 방식인 AWS Lambda입니다.