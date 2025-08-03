[이전 학습](../2-Understand-the-purpose-of-Terraform/2b-Explain-the-benefits-of-state.md) | [챕터 목록](./README.md) | [전체 목록](../README.md) | [다음 학습](./3b-Describe-plugin-based-architecture.md)

---

# 3a. Terraform 프로바이더 설치 및 버전 관리

Terraform은 프로바이더(Provider)라는 플러그인을 통해 다양한 클라우드 서비스(AWS, GCP, Azure 등) 및 SaaS 플랫폼과 상호작용합니다. 안정적이고 예측 가능한 인프라 관리를 위해서는 프로젝트에 필요한 프로바이더를 명시하고, 그 버전을 올바르게 관리하는 것이 매우 중요합니다.

## `required_providers` 설정 블록

Terraform 구성에서 사용할 프로바이더와 그 버전 제약 조건은 `terraform` 블록 안의 `required_providers` 블록에 명시합니다.

*   **`source`**: Terraform이 프로바이더를 다운로드할 레지스트리의 주소입니다. 일반적으로 `[<HOSTNAME>/]<NAMESPACE>/<TYPE>` 형식을 따릅니다. 예를 들어, HashiCorp가 관리하는 공식 AWS 프로바이더의 `source`는 `hashicorp/aws` 입니다.
*   **`version`**: 사용할 프로바이더의 버전을 지정합니다. 버전 제약 조건을 사용하여 특정 버전, 최소 버전, 버전 범위 등을 명시할 수 있습니다.

### Terraform 코드 예시

```terraform
terraform {
  required_providers {
    # HashiCorp 공식 AWS 프로바이더를 사용하며,
    # 버전은 5.0 이상이어야 함 (5.x 버전대)
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
    
    # HashiCorp 공식 Google Cloud 프로바이더를 사용하며,
    # 버전은 5.2.0으로 고정
    google = {
      source  = "hashicorp/google"
      version = "5.2.0"
    }
    
    # 커뮤니티에서 만든 Random 프로바이더를 사용하며,
    # 버전은 3.5.0 이상 4.0.0 미만
    random = {
        source = "hashicorp/random"
        version = ">= 3.5.0, < 4.0.0"
    }
  }
}

# AWS 프로바이더 설정
provider "aws" {
  region = "us-west-2"
}

# GCP 프로바이더 설정
provider "google" {
  project = "my-gcp-project"
  region  = "us-central1"
}
```

**버전 제약 조건 연산자:**

| 연산자 | 설명 | 예시 |
|---|---|---|
| `=` | 정확한 버전 일치 | `version = "1.2.3"` |
| `!=` | 특정 버전 제외 | `version != "1.2.3"` |
| `>`, `>=`, `<`, `<=` | 버전 비교 | `version > "1.2.3"` |
| `~>` | 비관적 제약(Pessimistic Constraint) | `~> 1.2.3`은 `>= 1.2.3, < 1.3.0`와 같음<br>`~> 1.2`는 `>= 1.2.0, < 2.0.0`와 같음 |


## 의존성 잠금 파일 (`.terraform.lock.hcl`)

`terraform init` 명령을 실행하면, Terraform은 `required_providers` 블록에 명시된 버전 제약 조건을 만족하는 프로바이더를 다운로드합니다. 그리고 이때 다운로드한 **정확한 프로바이더 버전과 해시(hash) 정보를 `.terraform.lock.hcl` 파일에 기록**합니다.

이 파일을 "의존성 잠금 파일(Dependency Lock File)"이라고 부릅니다.

### 잠금 파일의 주요 이점

1.  **일관성 있는 실행 환경**: 팀원들이나 CI/CD 파이프라인 등 어디에서 `terraform init`을 실행하더라도, 잠금 파일에 기록된 동일한 버전의 프로바이더를 사용하게 됩니다. 이를 통해 "제 컴퓨터에서는 됐는데..."와 같은 문제를 방지하고, 항상 예측 가능한 실행을 보장합니다.
2.  **보안**: 잠금 파일에는 프로바이더 패키지의 해시값이 포함되어 있습니다. `terraform init` 시 다운로드한 프로바이더의 해시가 잠금 파일의 해시와 일치하는지 검증하여, 의도치 않거나 악의적인 프로바이더가 설치되는 것을 방지합니다.

`.terraform.lock.hcl` 파일은 자동으로 생성되고 관리되므로 **직접 수정해서는 안 되며**, Git과 같은 버전 관리 시스템에 **반드시 포함하여 모든 팀원이 공유**해야 합니다.

### 프로바이더 업그레이드

프로바이더 버전을 업그레이드하려면 다음 두 단계를 따릅니다.

1.  `required_providers` 블록의 `version` 제약 조건을 수정합니다. (예: `~> 5.0` -> `~> 5.1`)
2.  `terraform init -upgrade` 명령을 실행하여 수정된 제약 조건에 맞는 최신 버전을 다운로드하고 `.terraform.lock.hcl` 파일을 업데이트합니다.

---

## 예상 문제

1.  **Terraform 구성에서 사용할 프로바이더와 그 버전을 지정하는 블록은 무엇입니까?**<br>
    a. `provider "aws"`<br>
    b. `resource "aws_instance"`<br>
    c. `terraform { required_providers { ... } }`<br>
    d. `variable "provider_version"`<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>c</p>
    </details><br>

2.  **`version = "~> 4.17"` 와 같은 버전 제약 조건의 의미로 올바른 것은 무엇입니까?**<br>
    a. 4.17.0 버전을 포함한 모든 4.x 버전대<br>
    b. 정확히 4.17.0 버전<br>
    c. 4.17.0 이상이면서 4.18.0 미만인 버전 (>= 4.17.0, < 4.18.0)<br>
    d. 4.17.0 이상이면서 5.0.0 미만인 버전 (>= 4.17.0, < 5.0.0)<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>d. `~>` 연산자는 마지막으로 명시된 버전 자리를 올리는 것을 허용합니다. `~> 4.17`은 `>= 4.17.0` 이면서 `< 5.0.0` 인 버전을 의미합니다.</p>
    </details><br>

3.  **`.terraform.lock.hcl` 파일의 주된 역할은 무엇입니까?**<br>
    a. Terraform 상태 정보를 저장한다.<br>
    b. 프로바이더의 자격 증명을 암호화하여 저장한다.<br>
    c. `terraform init` 시 설치할 정확한 프로바이더 버전과 해시를 기록하여 일관성을 보장한다.<br>
    d. `terraform plan`의 결과를 캐싱하여 성능을 높인다.<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>c</p>
    </details><br>

4.  **팀원들과 협업할 때 `.terraform.lock.hcl` 파일을 어떻게 다루어야 합니까?**<br>
    a. `.gitignore`에 추가하여 각자 다른 버전을 사용하도록 한다.<br>
    b. Git과 같은 버전 관리 시스템에 커밋하여 모든 팀원이 동일한 버전을 공유하도록 한다.<br>
    c. 수동으로 수정하여 원하는 프로바이더 버전을 강제한다.<br>
    d. `terraform init` 실행 전에 항상 삭제한다.<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>b</p>
    </details><br>

5.  **`terraform init` 실행 시 어떤 일이 발생합니까? (가장 올바른 설명)**<br>
    a. 인프라를 생성하거나 변경한다.<br>
    b. 구성 파일의 문법 오류를 검사한다.<br>
    c. 백엔드를 초기화하고, `required_providers`에 명시된 프로바이더를 다운로드하며, `.terraform.lock.hcl`을 생성/업데이트한다.<br>
    d. 상태 파일을 원격 백엔드로 푸시한다.<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>c</p>
    </details><br>

6.  **`required_providers` 블록에 프로바이더 `source`를 명시하는 주된 이유는 무엇입니까?**<br>
    a. 프로바이더의 별칭을 만들기 위해<br>
    b. Terraform이 어떤 레지스트리에서 해당 프로바이더를 찾아야 하는지 알려주기 위해<br>
    c. 프로바이더의 라이선스를 명시하기 위해<br>
    d. 프로바이더와 관련된 리소스만 필터링하기 위해<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>b</p>
    </details><br>

7.  **기존에 사용하던 프로바이더의 버전을 업그레이드하고 싶을 때 올바른 명령어는 무엇입니까?**<br>
    a. `terraform init`<br>
    b. `terraform init -upgrade`<br>
    c. `terraform providers upgrade`<br>
    d. `terraform refresh`<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>b</p>
    </details><br>

8.  **다음 중 유효하지 않은 버전 제약 조건은 무엇입니까?**<br>
    a. `version = "1.1.7"`<br>
    b. `version = ">= 1.0.0, < 2.0.0"`<br>
    c. `version = "!= 1.2.0"`<br>
    d. `version = "1.0.0 || 2.0.0"`<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>d. Terraform은 OR(||) 연산자를 버전 제약 조건에서 지원하지 않습니다.</p>
    </details><br>

9.  **`.terraform.lock.hcl` 파일에 기록되는 정보가 아닌 것은 무엇입니까?**<br>
    a. 프로바이더 버전 제약 조건 (예: `~> 5.0`)<br>
    b. 선택된 프로바이더의 정확한 버전 (예: `5.1.0`)<br>
    c. 프로바이더 패키지의 해시값<br>
    d. 프로바이더가 지원하는 운영체제 및 아키텍처<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>a. 버전 제약 조건은 `.tf` 파일에만 존재하며, 잠금 파일에는 이를 만족하는 '선택된' 버전이 기록됩니다.</p>
    </details><br>

10. **`terraform init` 명령이 `.terraform.lock.hcl` 파일을 사용하는 방식에 대한 설명으로 가장 올바른 것은 무엇입니까?**<br>
    a. 잠금 파일이 있으면 항상 무시하고 `required_providers` 정의에 따라 새로 다운로드한다.<br>
    b. 잠금 파일이 있으면, 파일에 명시된 버전과 해시를 사용하여 프로바이더를 설치하여 일관성을 유지한다.<br>
    c. 잠금 파일은 프로바이더 다운로드에 영향을 주지 않고, 오직 백엔드 초기화에만 사용된다.<br>
    d. 잠금 파일이 있으면 오류를 발생시키며 중단된다.<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>b</p>
    </details><br>

---

[이전 학습](../2-Understand-the-purpose-of-Terraform/2b-Explain-the-benefits-of-state.md) | [챕터 목록](./README.md) | [전체 목록](../../README.md) | [다음 학습](./3b-Describe-plugin-based-architecture.md)
