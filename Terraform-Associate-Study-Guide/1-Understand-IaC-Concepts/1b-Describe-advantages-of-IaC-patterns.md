[이전 학습](./1a-Explain-what-IaC-is.md) | [챕터 목록](./README.md) | [전체 목록](../README.md) | [다음 학습](../2-Understand-the-purpose-of-Terraform/2a-Explain-multi-cloud-and-provider-agnostic-benefits.md)

---

# 1b. IaC(Infrastructure as Code) 패턴의 이점

## IaC 패턴의 주요 이점

IaC(Infrastructure as Code)를 사용하면 인프라 관리를 효율적이고 안정적으로 만들 수 있습니다. 주요 이점은 다음과 같습니다.

1.  **자동화를 통한 속도 및 효율성 향상**
    *   IaC는 인프라 생성, 수정, 삭제 과정을 자동화하여 수동으로 작업할 때보다 훨씬 빠르게 인프라를 배포할 수 있습니다. 이를 통해 개발자는 몇 시간이 아닌 몇 분 만에 전체 환경을 구축할 수 있습니다.

2.  **일관성 및 신뢰성 확보**
    *   모든 인프라 구성이 코드로 정의되므로, 개발, 스테이징, 프로덕션 환경에 걸쳐 동일한 구성을 일관되게 적용할 수 있습니다. 이는 "내 컴퓨터에서는 됐는데…"와 같은 문제를 방지하고, 수동 작업으로 인한 인적 오류를 줄여줍니다.

3.  **버전 관리 및 협업 강화**
    *   인프라 코드를 Git과 같은 버전 관리 시스템(VCS)에 저장하면, 모든 변경 사항을 추적하고 검토할 수 있습니다. 누가, 무엇을, 왜 변경했는지 명확하게 알 수 있으며, 문제가 발생했을 때 이전 버전으로 쉽게 되돌릴 수 있습니다. 이는 팀원 간의 협업을 원활하게 합니다.

4.  **재사용성을 통한 표준화**
    *   반복적으로 사용되는 인프라 구성(예: 웹 서버, 데이터베이스)을 "모듈(Module)"이라는 단위로 캡슐화하여 재사용할 수 있습니다. 이를 통해 조직의 모범 사례를 표준화하고, 새로운 환경을 더 빠르고 안정적으로 구축할 수 있습니다.

5.  **비용 절감**
    *   인프라를 코드화하면 리소스를 필요할 때만 생성하고, 사용하지 않을 때는 쉽게 제거할 수 있습니다. `terraform destroy`와 같은 명령어를 사용하여 테스트 환경이나 임시 환경을 빠르게 정리함으로써 불필요한 클라우드 비용을 절감할 수 있습니다.

## Terraform 코드 예시: 모듈을 사용한 재사용성

다음은 Terraform 모듈을 사용하여 웹 서버를 배포하는 예시입니다. `app-server`라는 모듈을 호출하여 EC2 인스턴스를 생성합니다. 이 모듈은 다른 프로젝트에서도 동일하게 재사용될 수 있습니다.

```terraform
# ./main.tf

# AWS 프로바이더 설정
provider "aws" {
  region = "us-east-1"
}

# "app-server" 모듈을 사용하여 인스턴스 생성
module "web_server" {
  source        = "./modules/app-server" # 로컬 모듈 경로
  instance_type = "t2.micro"
  server_name   = "ProductionServer"
}

module "staging_server" {
  source        = "./modules/app-server"
  instance_type = "t2.nano"
  server_name   = "StagingServer"
}
```

```terraform
# ./modules/app-server/main.tf

# 입력 변수 선언
variable "instance_type" {}
variable "server_name" {}

# EC2 인스턴스 리소스 정의
resource "aws_instance" "server" {
  ami           = "ami-0c55b159cbfafe1f0" # 예시 AMI ID
  instance_type = var.instance_type

  tags = {
    Name = var.server_name
  }
}
```

위 예시처럼 모듈을 사용하면 프로덕션 환경과 스테이징 환경을 동일한 구성으로, 하지만 다른 사양(인스턴스 유형)으로 쉽게 배포할 수 있습니다. 이는 코드의 재사용성을 높이고 일관성을 유지하는 IaC의 핵심 패턴입니다.

---

## 예상 문제

1.  **IaC를 사용했을 때 얻을 수 있는 가장 큰 장점 중 하나는 무엇입니까?**<br>
    a. 모든 인프라를 물리적으로 직접 제어할 수 있다.<br>
    b. 인프라 변경 속도를 늦춰 안정성을 높인다.<br>
    c. 수동 작업을 줄여 인적 오류를 감소시킨다.<br>
    d. 클라우드 제공업체에 대한 종속성을 완전히 제거한다.<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>c</p>
    </details><br>

2.  **인프라 코드를 Git과 같은 버전 관리 시스템으로 관리할 때의 이점이 아닌 것은 무엇입니까?**<br>
    a. 변경 이력 추적<br>
    b. 팀원 간의 협업 용이성<br>
    c. 문제 발생 시 이전 상태로의 빠른 복구<br>
    d. 클라우드 리소스 비용 자동 계산<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>d</p>
    </details><br>

3.  **Terraform에서 "모듈(Module)"을 사용하는 주된 이유는 무엇입니까?**<br>
    a. Terraform 바이너리 버전을 고정하기 위해<br>
    b. 코드를 재사용하고 인프라 패턴을 표준화하기 위해<br>
    c. 민감한 정보를 안전하게 저장하기 위해<br>
    d. 단일 파일에서만 작업하도록 강제하기 위해<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>b</p>
    </details><br>

4.  **동일한 코드를 여러 번 적용해도 결과가 동일하게 유지되는 IaC의 특징을 무엇이라고 합니까?**<br>
    a. 가변성(Mutability)<br>
    b. 멱등성(Idempotency)<br>
    c. 절차성(Procedural)<br>
    d. 종속성(Dependency)<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>b</p>
    </details><br>

5.  **IaC를 통해 개발, 스테이징, 프로덕션 환경의 구성을 일관되게 유지하는 것을 무엇이라고 합니까?**<br>
    a. 환경 패리티(Environment Parity)<br>
    b. 지속적 통합(Continuous Integration)<br>
    c. 수직적 확장(Vertical Scaling)<br>
    d. 블루/그린 배포(Blue/Green Deployment)<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>a</p>
    </details><br>

6.  **`terraform destroy` 명령어를 사용하여 얻을 수 있는 주요 이점은 무엇입니까?**<br>
    a. 인프라 구성을 자동으로 문서화한다.<br>
    b. 사용하지 않는 리소스를 정리하여 비용을 절감한다.<br>
    c. 새로운 Terraform 버전을 자동으로 설치한다.<br>
    d. 인프라의 보안 취약점을 분석한다.<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>b</p>
    </details><br>

7.  **IaC 패턴이 DevOps 문화에 기여하는 방식은 무엇입니까?**<br>
    a. 개발팀과 운영팀의 업무를 완전히 분리시킨다.<br>
    b. 인프라를 코드로 취급하여 개발자와 운영자 간의 협업을 촉진한다.<br>
    c. 모든 운영 업무를 개발자에게 이전시킨다.<br>
    d. 코드 없는(No-code) 솔루션을 장려한다.<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>b</p>
    </details><br>

8.  **다음 중 IaC의 "선언적" 접근 방식의 장점은 무엇입니까?**<br>
    a. 사용자가 모든 작업 단계를 직접 코딩해야 한다.<br>
    b. 최종 목표 상태만 정의하면 되므로 복잡성이 줄어든다.<br>
    c. 기존 인프라를 절대 변경할 수 없다.<br>
    d. 오직 새로운 리소스 생성만 가능하다.<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>b</p>
    </details><br>

9.  **인프라 변경 사항을 배포하기 전에 `terraform plan`을 실행하는 것의 가치는 무엇입니까?**<br>
    a. 코드를 자동으로 포맷팅해준다.<br>
    b. 실제 변경이 적용되기 전에 예상되는 변경 사항을 검토하여 실수를 방지한다.<br>
    c. 인프라 비용을 즉시 결제한다.<br>
    d. 팀원들에게 이메일로 변경 계획을 자동으로 전송한다.<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>b</p>
    </details><br>

10. **IaC를 사용하면 여러 클라우드 제공업체에 걸쳐 인프라를 관리하는 것이 어떻게 더 쉬워집니까?**<br>
    a. 각 클라우드마다 완전히 다른 프로그래밍 언어를 사용해야 한다.<br>
    b. Terraform과 같은 도구가 여러 제공업체를 지원하는 단일 워크플로우를 제공한다.<br>
    c. 클라우드 간 데이터 마이그레이션이 자동으로 수행된다.<br>
    d. 특정 클라우드 제공업체에 대한 전문 지식이 전혀 필요 없게 된다.<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>b</p>
    </details><br>

---

[이전 학습](./1a-Explain-what-IaC-is.md) | [챕터 목록](./README.md) | [전체 목록](../../README.md) | [다음 학습](../../2-Understand-the-purpose-of-Terraform/README.md)