[이전 학습](../1-Understand-IaC-Concepts/1b-Describe-advantages-of-IaC-patterns.md) | [챕터 목록](./README.md) | [전체 목록](../README.md) | [다음 학습](./2b-Explain-the-benefits-of-state.md)

---

# 2a. 멀티-클라우드 및 프로바이더 독립적인 이점 설명

## 멀티-클라우드(Multi-Cloud)의 개념과 이점

멀티-클라우드는 단일 클라우드 제공업체(예: AWS, Azure, GCP)에만 의존하는 대신, 두 개 이상의 클라우드 플랫폼을 동시에 사용하여 인프라를 구축하고 운영하는 전략입니다.

주요 이점은 다음과 같습니다.

*   **공급업체 종속성(Vendor Lock-in) 방지**: 특정 클라우드 제공업체의 기술, 가격 정책, 서비스 중단에 완전히 종속되는 것을 피할 수 있습니다. 이를 통해 비즈니스 유연성을 확보하고 협상력을 높일 수 있습니다.
*   **장애 복구 및 가용성 향상**: 하나의 클라우드 제공업체에서 장애가 발생하더라도 다른 클라우드에서 서비스를 계속 운영하여 전체 시스템의 다운타임을 최소화하고 안정성을 높일 수 있습니다. (예: AWS 장애 시 GCP에서 서비스 재개)
*   **최적의 서비스 선택**: 각 클라우드 제공업체는 특정 영역에서 강점을 가집니다. (예: A사는 AI/ML, B사는 데이터 분석) 워크로드의 특성에 가장 적합하고 비용 효율적인 서비스를 자유롭게 선택하여 조합할 수 있습니다.

## 프로바이더 독립적(Provider-Agnostic)인 Terraform의 역할

멀티-클라우드 전략은 강력하지만, 각 클라우드 제공업체마다 서로 다른 API, 도구, 워크플로우를 가지고 있어 운영 복잡성이 크게 증가한다는 단점이 있습니다.

Terraform은 **프로바이더(Provider)** 라는 플러그인 아키텍처를 통해 이러한 문제를 해결합니다. Terraform은 AWS, Azure, GCP, Kubernetes 등 수백 개의 다양한 인프라 제공업체를 위한 프로바이더를 지원합니다.

사용자는 어떤 클라우드를 사용하든 상관없이 **동일한 HCL(HashiCorp Configuration Language) 문법과 `init` -> `plan` -> `apply` 라는 일관된 워크플로우**를 사용하여 인프라를 관리할 수 있습니다. 이것이 바로 Terraform이 "프로바이더에 독립적(Provider-Agnostic)"이라고 불리는 이유입니다.

## Terraform 코드 예시: AWS와 GCP에 동시 배포

다음 코드는 Terraform을 사용하여 AWS에는 EC2 인스턴스를, GCP에는 Compute Engine 인스턴스를 동시에 생성하는 예시입니다. 두 개의 다른 클라우드 리소스를 하나의 Terraform 구성 파일에서 동일한 방식으로 관리할 수 있습니다.

```terraform
# 1. AWS 프로바이더 설정
provider "aws" {
  region = "us-west-2"
}

# 2. GCP 프로바이더 설정
provider "google" {
  project = "my-gcp-project-id"
  region  = "us-central1"
}

# 3. AWS에 EC2 인스턴스 생성
resource "aws_instance" "web_aws" {
  provider      = aws
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"

  tags = {
    Name = "WebServer-AWS"
  }
}

# 4. GCP에 Compute Engine 인스턴스 생성
resource "google_compute_instance" "web_gcp" {
  provider     = google
  name         = "web-server-gcp"
  machine_type = "e2-micro"
  zone         = "us-central1-a"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-11"
    }
  }

  network_interface {
    network = "default"
  }
}
```

이처럼 Terraform을 사용하면 여러 클라우드에 걸친 복잡한 인프라를 단일 워크플로우로 통합하여 효율적으로 관리할 수 있습니다.

---

## 예상 문제

1.  **멀티-클라우드 전략의 주요 이점으로 가장 적절한 것은 무엇입니까?**<br>
    a. 단일 클라우드 제공업체에 대한 기술 전문성 강화<br>
    b. 인프라 관리의 복잡성 감소<br>
    c. 특정 공급업체에 대한 종속성(Lock-in) 회피 및 유연성 증대<br>
    d. 모든 클라우드에서 동일한 성능 보장<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>c</p>
    </details><br>

2.  **Terraform이 "프로바이더에 독립적(Provider-Agnostic)"이라고 불리는 이유는 무엇입니까?**<br>
    a. Terraform은 오직 하나의 클라우드 제공업체만 지원하기 때문에<br>
    b. 클라우드 제공업체의 종류와 상관없이 동일한 언어와 워크플로우를 사용하기 때문에<br>
    c. 클라우드 제공업체의 API를 직접 호출할 필요가 없기 때문에<br>
    d. Terraform 코드가 모든 클라우드에서 자동으로 변환되기 때문에<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>b</p>
    </details><br>

3.  **Terraform에서 여러 클라우드 제공업체를 사용하기 위해 가장 먼저 설정해야 하는 구성 블록은 무엇입니까?**<br>
    a. `resource`\
    b. `variable`\
    c. `output`\
    d. `provider`<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>d</p>
    </details><br>

4.  **다음 중 멀티-클라우드 환경에서 Terraform을 사용했을 때의 장점이 아닌 것은 무엇입니까?**<br>
    a. 여러 클라우드에 걸친 인프라를 단일 구성으로 관리<br>
    b. 클라우드 간 리소스 종속성 관리 용이\
    c. 각 클라우드 제공업체의 고유한 기능을 전혀 사용할 수 없게 됨\
    d. 일관된 명령(`plan`, `apply`)으로 모든 클라우드 인프라 관리<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>c</p>
    </details><br>

5.  **Terraform의 프로바이더(Provider)는 어떤 역할을 합니까?**<br>
    a. Terraform 코드의 문법을 검사하는 역할<br>
    b. Terraform과 특정 클라우드 API 간의 통신을 담당하는 플러그인<br>
    c. 인프라 비용을 계산하는 역할<br>
    d. Terraform 상태 파일을 암호화하는 역할<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>b</p>
    </details><br>

6.  **한 Terraform 구성에서 AWS와 Azure 리소스를 동시에 생성할 수 있습니까?**<br>
    a. 아니요, 하나의 구성 파일은 하나의 제공업체만 대상으로 할 수 있습니다.<br>
    b. 예, 각 리소스 블록에 `provider` 메타-인수를 지정하여 가능합니다.<br>
    c. 아니요, AWS와 Azure는 호환되지 않아 불가능합니다.<br>
    d. 예, 하지만 별도의 `terraform apply` 명령을 실행해야 합니다.<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>b</p>
    </details><br>

7.  **기업이 멀티-클라우드를 채택하는 이유 중 하나인 '최적의 서비스 선택'은 무엇을 의미합니까?**<br>
    a. 가장 저렴한 클라우드 제공업체 하나만 선택하는 것<br>
    b. 각기 다른 클라우드의 강점을 조합하여 최상의 시스템을 구축하는 것<br>
    c. 가장 많은 종류의 서비스를 제공하는 클라우드를 선택하는 것<br>
    d. 모든 워크로드를 동일한 서비스에 배포하는 것<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>b</p>
    </details><br>

8.  **Terraform의 어떤 명령어가 여러 프로바이더 플러그인을 다운로드하고 설치합니까?**<br>
    a. `terraform plan`\
    b. `terraform apply`\
    c. `terraform init`\
    d. `terraform validate`<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>c</p>
    </details><br>

9.  **멀티-클라우드 환경의 가장 큰 단점은 무엇이며, Terraform은 이를 어떻게 해결합니까?**<br>
    a. 단점: 비용 증가 / 해결책: 가장 저렴한 리소스만 사용<br>
    b. 단점: 운영 복잡성 증가 / 해결책: 일관된 워크플로우 제공\
    c. 단점: 보안 취약 / 해결책: 강력한 암호화 \
    d. 단점: 낮은 성능 / 해결책: 리소스 자동 확장<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>b</p>
    </details><br>

10. **리소스 블록 내에서 `provider = aws` 와 같이 명시하는 메타-인수의 역할은 무엇입니까?**<br>
    a. 리소스의 이름을 지정합니다.<br>
    b. 리소스가 의존하는 다른 리소스를 지정합니다.<br>
    c. 기본 프로바이더 구성이 아닌, 특정 프로바이더 구성을 사용하도록 지정합니다.<br>
    d. 리소스 생성 개수를 지정합니다.<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>c</p>
    </details><br>

---

[이전 학습](../1-Understand-IaC-Concepts/1b-Describe-advantages-of-IaC-patterns.md) | [챕터 목록](./README.md) | [전체 목록](../../README.md) | [다음 학습](./2b-Explain-the-benefits-of-state.md)
