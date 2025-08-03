[챕터 목록](./README.md) | [전체 목록](../README.md) | [다음 학습](./1b-Describe-advantages-of-IaC-patterns.md)

---

# 1a. IaC(Infrastructure as Code)란 무엇인가?

## IaC(Infrastructure as Code)의 정의

IaC(Infrastructure as Code)는 코드를 사용하여 인프라를 정의하고 관리하는 방식입니다. 기존에는 데이터 센터의 물리적 서버나 가상 머신을 구성하기 위해 수동으로 설정하거나 스크립트를 작성했지만, IaC는 구성 파일을 통해 인프라를 프로비저닝하고 관리합니다.

이러한 접근 방식은 개발자가 애플리케이션 코드를 작성하는 것과 유사하게 인프라를 처리할 수 있게 해주며, 다음과 같은 이점을 제공합니다.

*   **버전 관리**: 인프라 구성을 Git과 같은 버전 관리 시스템으로 추적하여 변경 사항을 쉽게 되돌리거나 협업할 수 있습니다.
*   **자동화**: 인프라 생성 및 변경 프로세스를 자동화하여 수동 작업으로 인한 실수를 줄이고 효율성을 높입니다.
*   **재사용성**: 공통된 인프라 패턴을 모듈화하여 여러 환경에서 재사용할 수 있습니다.

Terraform은 IaC를 구현하는 대표적인 도구 중 하나로, 사람이 읽기 쉬운 선언적 언어(HCL)를 사용하여 인프라의 "최종 상태(desired state)"를 정의합니다.

## Terraform 코드 예시

다음은 간단한 Terraform 코드 예시입니다. 이 코드는 AWS에 `t2.micro` 인스턴스 유형의 EC2 인스턴스를 생성하는 것을 정의합니다.

```terraform
# AWS 프로바이더 설정
provider "aws" {
  region = "us-west-2"
}

# EC2 인스턴스 리소스 정의
resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0" # 예시 AMI ID
  instance_type = "t2.micro"

  tags = {
    Name = "ExampleInstance"
  }
}
```

위 코드에서 사용자는 "어떤 인프라를 원하는지"를 선언하기만 하면, Terraform이 알아서 해당 리소스를 생성하는 데 필요한 API 호출을 처리합니다.

---

## 예상 문제

1.  **IaC(Infrastructure as Code)의 가장 적절한 정의는 무엇입니까?**<br>
    a. 스크립트를 사용하여 인프라를 수동으로 구성하는 방법<br>
    b. 코드를 사용하여 인프라를 프로비저닝하고 관리하는 프로세스<br>
    c. 그래픽 사용자 인터페이스(GUI)를 통해 클라우드 리소스를 관리하는 것<br>
    d. 물리적 하드웨어를 직접 설정하는 전통적인 IT 방식<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>b</p>
    </details><br>

2.  **다음 중 IaC의 핵심 이점이 아닌 것은 무엇입니까?**<br>
    a. 인프라 구성의 버전 관리<br>
    b. 수동 배포를 통한 보안 강화<br>
    c. 인프라 프로비저닝 자동화<br>
    d. 인프라 패턴의 재사용성<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>b</p>
    </details><br>

3.  **Terraform에서 사용되는 기본 언어는 무엇입니까?**<br>
    a. Python<br>
    b. YAML<br>
    c. HCL (HashiCorp Configuration Language)<br>
    d. JSON<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>c</p>
    </details><br>

4.  **Terraform 구성 파일에서 "리소스(resource)" 블록의 역할은 무엇입니까?**<br>
    a. Terraform 버전 설정<br>
    b. 클라우드 제공업체(provider) 지정<br>
    c. 생성하거나 관리할 인프라 객체(예: EC2 인스턴스) 정의<br>
    d. 변수 값 할당<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>c</p>
    </details><br>

5.  **"선언적(declarative)" IaC 접근 방식의 의미는 무엇입니까?**<br>
    a. 인프라를 생성하기 위한 단계별 절차를 명시해야 한다.<br>
    b. 원하는 최종 상태를 정의하면 도구가 알아서 구현 방법을 결정한다.<br>
    c. 기존 인프라를 수정할 수 없음을 의미한다.<br>
    d. 오직 스크립트 기반의 명령형 코드만 사용 가능하다.<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>b</p>
    </details><br>

6.  **IaC를 사용하면 인프라 구성을 어떻게 추적하고 협업할 수 있습니까?**<br>
    a. 이메일을 통해 구성 파일을 공유한다.<br>
    b. 위키(Wiki) 페이지에 변경 사항을 수동으로 기록한다.<br>
    c. Git과 같은 버전 관리 시스템(VCS)을 사용한다.<br>
    d. 공유 네트워크 드라이브에 파일을 저장한다.<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>c</p>
    </details><br>

7.  **Terraform 코드 예시에서 `provider "aws"` 블록의 목적은 무엇입니까?**<br>
    a. 생성될 EC2 인스턴스의 유형을 지정한다.<br>
    b. 사용할 AWS 리전과 같은 프로바이더 관련 설정을 구성한다.<br>
    c. 인스턴스에 부착할 태그를 정의한다.<br>
    d. Terraform의 실행 모드를 설정한다.<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>b</p>
    </details><br>

8.  **다음 중 IaC가 해결하고자 하는 주요 문제가 아닌 것은 무엇입니까?**<br>
    a. 수동 프로비저닝으로 인한 인적 오류<br>
    b. 여러 환경 간의 구성 불일치<br>
    c. 애플리케이션 소스 코드의 품질 저하<br>
    d. 느리고 비효율적인 인프라 배포 프로세스<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>c</p>
    </details><br>

9.  **Terraform은 인프라를 어떻게 "상태(state)"를 관리합니까?**<br>
    a. 매번 실행 시 인프라를 처음부터 다시 생성한다.<br>
    b. 구성 파일에 직접 현재 상태를 기록한다.<br>
    c. 별도의 상태 파일(.tfstate)을 사용하여 실제 인프라와 구성을 매핑한다.<br>
    d. 클라우드 제공업체의 콘솔에만 의존하여 상태를 확인한다.<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>c</p>
    </details><br>

10. **IaC를 도입했을 때 얻을 수 있는 가장 큰 문화적 변화는 무엇이라고 생각할 수 있습니까?**<br>
    a. 개발팀과 운영팀의 역할이 완전히 분리된다.<br>
    b. 인프라 관리가 더 이상 중요하지 않게 된다.<br>
    c. 개발자와 운영자가 코드를 중심으로 협업하는 DevOps 문화가 촉진된다.<br>
    d. 모든 IT 운영이 아웃소싱된다.<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>c</p>
    </details><br>

---

[챕터 목록](./README.md) | [전체 목록](../../README.md) | [다음 학습](./1b-Describe-advantages-of-IaC-patterns.md)