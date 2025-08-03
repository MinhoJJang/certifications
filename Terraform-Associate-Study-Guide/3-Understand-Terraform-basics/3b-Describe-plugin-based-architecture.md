[이전 학습](./3a-Install-and-version-Terraform-providers.md) | [챕터 목록](./README.md) | [전체 목록](../README.md) | [다음 학습](./3c-Write-Terraform-configuration-using-multiple-providers.md)

---

# 3b. Terraform의 플러그인 기반 아키텍처 설명

Terraform의 가장 큰 특징 중 하나는 **플러그인 기반 아키텍처(Plugin-Based Architecture)**를 채택했다는 점입니다. 이 구조 덕분에 Terraform은 수많은 클라우드 제공업체, SaaS 플랫폼, 온프레미스 솔루션 등 거의 모든 종류의 API를 지원할 수 있는 강력한 확장성을 가집니다.

## Terraform Core와 플러그인

Terraform은 두 가지 주요 구성 요소로 나뉩니다.

1.  **Terraform Core**:
    *   Terraform의 핵심 실행 파일(`terraform`)입니다.
    *   주요 역할은 다음과 같습니다.
        *   Terraform 구성 파일(`.tf`)을 읽고 해석합니다.
        *   리소스 상태 파일을 관리합니다.
        *   리소스 의존성 그래프를 생성하여 실행 계획(`plan`)을 수립합니다.
        *   사용자와의 상호작용(CLI)을 담당합니다.
        *   플러그인과의 통신을 관리합니다.
    *   Terraform Core 자체는 AWS, GCP 등 특정 인프라를 어떻게 다뤄야 하는지 전혀 알지 못합니다.

2.  **플러그인 (Plugins)**:
    *   특정 서비스(예: AWS, GCP, Kubernetes, GitHub 등)의 API와 통신하는 역할을 담당하는 독립적인 실행 파일입니다.
    *   가장 일반적인 플러그인 유형은 **프로바이더(Provider)**입니다.
    *   각 프로바이더는 특정 서비스의 리소스를 생성(Create), 읽기(Read), 업데이트(Update), 삭제(Delete)하는 방법을 구현하고 있습니다. (CRUD)
    *   Terraform Core는 이 프로바이더 플러그인을 통해 특정 클라우드의 리소스를 실제로 관리하게 됩니다.

## Core와 플러그인의 통신 방식

Terraform Core와 프로바이더 플러그인은 **RPC(Remote Procedure Call)**를 통해 통신합니다. `terraform init` 시점에 Core는 필요한 프로바이더 플러그인을 다운로드하고, `plan`이나 `apply` 같은 명령이 실행되면 Core는 RPC를 사용하여 해당 프로바이더에게 "이 AWS 인스턴스를 생성해줘" 또는 "이 GCP 버킷의 현재 상태를 알려줘" 와 같은 요청을 보냅니다.

이러한 분리 구조는 다음과 같은 큰 이점을 가집니다.

*   **확장성 (Extensibility)**: 새로운 클라우드 서비스나 API가 등장하더라도 Terraform Core를 수정할 필요 없이, 해당 서비스를 위한 프로바이더 플러그인만 개발하면 Terraform에서 즉시 지원할 수 있습니다.
*   **관심사 분리 (Separation of Concerns)**: Terraform Core는 인프라 관리의 일반적인 워크플로우(plan, apply, state 등)에만 집중하고, 각 프로바이더는 자신이 담당하는 서비스의 API 세부 사항에만 집중할 수 있습니다. 이로 인해 개발 및 유지보수가 훨씬 용이해집니다.
*   **독립적인 버전 관리**: Terraform Core의 버전과 각 프로바이더의 버전을 독립적으로 관리하고 업그레이드할 수 있습니다. (예: Terraform v1.5.0에서 AWS 프로바이더 v4.x와 v5.x를 모두 사용 가능)
*   **커뮤니티 활성화**: HashiCorp뿐만 아니라 수많은 기업과 개인이 직접 프로바이더를 개발하여 Terraform 생태계에 기여할 수 있습니다.

### Terraform 코드 예시

다음 코드는 Terraform Core가 어떻게 `aws`와 `random`이라는 두 개의 서로 다른 프로바이더 플러그인을 사용하여 작업을 수행하는지 보여줍니다. Core는 두 프로바이더에게 각자의 역할을 지시할 뿐, EC2 인스턴스나 무작위 문자열을 직접 생성하는 방법을 알지 못합니다.

```terraform
# 사용할 프로바이더 플러그인 정의
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
    random = {
      source  = "hashicorp/random"
      version = "~> 3.5"
    }
  }
}

# 1. 'aws' 프로바이더에게 EC2 인스턴스 생성을 요청
resource "aws_instance" "web" {
  provider      = aws
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
}

# 2. 'random' 프로바이더에게 무작위 pet 이름을 생성하도록 요청
resource "random_pet" "server_name" {
  length = 2
}
```

이처럼 플러그인 아키텍처는 Terraform을 단순한 도구가 아닌, 모든 인프라를 코드로 관리할 수 있는 강력한 "플랫폼"으로 만들어주는 핵심적인 개념입니다.

---

## 예상 문제

1.  **Terraform의 아키텍처에서 Terraform Core의 주요 책임이 아닌 것은 무엇입니까?**<br>
    a. 구성 파일(.tf) 파싱<br>
    b. 리소스 의존성 그래프 생성<br>
    c. AWS EC2 인스턴스 생성을 위한 API 직접 호출<br>
    d. 상태 파일(state) 관리<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>c. 특정 클라우드의 API를 직접 호출하는 것은 프로바이더 플러그인의 역할입니다.</p>
    </details><br>

2.  **특정 클라우드 서비스(예: Azure)의 리소스를 생성, 읽기, 업데이트, 삭제하는 로직을 구현하는 Terraform 구성 요소는 무엇입니까?**<br>
    a. Terraform Core<br>
    b. 상태 파일(State File)<br>
    c. 프로바이더 플러그인(Provider Plugin)<br>
    d. 의존성 잠금 파일(Lock File)<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>c</p>
    </details><br>

3.  **Terraform Core와 프로바이더 플러그인은 어떤 방식으로 통신합니까?**<br>
    a. 공유 메모리<br>
    b. RPC (Remote Procedure Call)<br>
    c. 직접적인 함수 호출<br>
    d. JSON 파일 교환<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>b</p>
    </details><br>

4.  **Terraform의 플러그인 기반 아키텍처가 제공하는 가장 큰 이점은 무엇입니까?**<br>
    a. Terraform 실행 속도 향상<br>
    b. Terraform 바이너리 파일 크기 감소<br>
    c. 새로운 기술 및 서비스를 지원하는 뛰어난 확장성<br>
    d. HCL 문법의 단순화<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>c</p>
    </details><br>

5.  **`terraform init` 명령을 실행할 때 플러그인과 관련하여 발생하는 일은 무엇입니까?**<br>
    a. 구성에 정의된 모든 리소스를 생성한다.<br>
    b. 구성에 필요한 프로바이더 플러그인을 다운로드하고 설치한다.<br>
    c. Terraform Core를 최신 버전으로 업그레이드한다.<br>
    d. 모든 프로바이더 플러그인을 삭제하고 재설치한다.<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>b</p>
    </details><br>

6.  **사용자가 직접 만든 Private Cloud를 Terraform으로 관리하고 싶을 때 가장 먼저 해야 할 일은 무엇입니까?**<br>
    a. Terraform Core의 소스 코드를 수정한다.<br>
    b. 해당 Private Cloud의 API와 통신할 수 있는 커스텀 프로바이더를 개발한다.<br>
    c. HashiCorp에 새로운 프로바이더 개발을 요청한다.<br>
    d. AWS 프로바이더를 수정하여 사용한다.<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>b</p>
    </details><br>

7.  **Terraform Core와 프로바이더의 버전이 독립적으로 관리될 수 있는 이유는 무엇입니까?**<br>
    a. 항상 같은 버전으로만 배포되기 때문에<br>
    b. Core와 플러그인이 분리되어 있고, RPC라는 표준화된 방식으로 통신하기 때문에<br>
    c. 프로바이더는 버전이 없기 때문에<br>
    d. 상태 파일이 버전 정보를 관리해주기 때문에<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>b</p>
    </details><br>

8.  **다음 중 프로바이더 플러그인의 예시로 가장 적절하지 않은 것은 무엇입니까?**<br>
    a. `hashicorp/aws`<br>
    b. `hashicorp/google`<br>
    c. `hashicorp/kubernetes`<br>
    d. `hashicorp/terraform`<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>d. `hashicorp/terraform`은 프로바이더가 아니라 Terraform Core 자체를 지칭합니다.</p>
    </details><br>

9.  **Terraform의 "관심사 분리" 원칙에 대한 설명으로 가장 올바른 것은 무엇입니까?**<br>
    a. 사용자는 인프라 구성과 상태 관리를 분리해서 생각해야 한다.<br>
    b. 개발 환경과 운영 환경의 구성을 분리해야 한다.<br>
    c. Terraform Core는 일반적인 워크플로우에, 프로바이더는 특정 API 연동에만 집중한다.<br>
    d. 네트워크 리소스와 컴퓨팅 리소스는 항상 다른 파일에 정의해야 한다.<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>c</p>
    </details><br>

10. **커뮤니티 프로바이더(Community Provider)란 무엇을 의미합니까?**<br>
    a. HashiCorp가 공식적으로 개발하고 유지보수하는 프로바이더<br>
    b. Terraform 유료 버전에서만 사용할 수 있는 프로바이더<br>
    c. Terraform 커뮤니티의 개인이나 기업이 개발하여 레지스트리에 등록한 프로바이더<br>
    d. 특정 지역에서만 사용할 수 있는 프로바이더<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>c</p>
    </details><br>

---

[이전 학습](./3a-Install-and-version-Terraform-providers.md) | [챕터 목록](./README.md) | [전체 목록](../../README.md) | [다음 학습](./3c-Write-Terraform-configuration-using-multiple-providers.md)
