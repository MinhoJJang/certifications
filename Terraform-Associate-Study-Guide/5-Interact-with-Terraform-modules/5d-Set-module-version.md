[이전 학습](./5c-Describe-variable-scope-within-modules-child-modules.md) | [챕터 목록](./README.md) | [전체 목록](../README.md) | [다음 학습](../6-Use-the-core-Terraform-workflow/README.md)

---

# 5d. 모듈 버전 설정하기

Terraform 프로바이더의 버전을 고정하여 예측 가능한 실행을 보장하는 것처럼, 원격 모듈의 버전을 명시적으로 설정하는 것 또한 안정적인 인프라 관리를 위해 매우 중요합니다.

모듈 버전을 고정하면, 모듈의 원본 소스가 변경되더라도 현재 구성이 영향을 받지 않도록 보호할 수 있습니다. 이를 통해 "의도치 않은 변경"을 방지하고 인프라 구성의 일관성과 안정성을 유지할 수 있습니다.

## `version` 인수 사용하기

Terraform 레지스트리(퍼블릭 또는 프라이빗)에서 가져오는 모듈의 경우, `module` 블록 내에 `version` 인수를 사용하여 원하는 버전 제약 조건을 명시할 수 있습니다.

`version` 인수는 프로바이더의 버전 제약 조건과 동일한 구문을 사용합니다.

| 연산자 | 설명 | 예시 |
|---|---|---|
| `=` | 정확한 버전 일치 (권장) | `version = "1.2.3"` |
| `!=` | 특정 버전 제외 | `version != "1.2.3"` |
| `>`, `>=`, `<`, `<=` | 버전 비교 | `version >= "1.2.0"` |
| `~>` | 비관적 제약(Pessimistic Constraint) | `~> 1.2.3`는 `>= 1.2.3, < 1.3.0`와 같음<br>`~> 1.2`는 `>= 1.2.0, < 2.0.0`와 같음 |

### 왜 버전 고정이 중요한가?

상상해보세요. 여러분이 사용하던 아주 유용한 공개 모듈의 개발자가 어느 날 새로운 기능(breaking change)을 추가하면서 기존의 입력 변수 이름을 바꾸거나 리소스 구조를 변경했습니다. 만약 여러분의 코드에 버전이 명시되어 있지 않다면, 다음에 `terraform init`을 실행했을 때 이 변경된 모듈 코드를 그대로 다운로드하게 될 것입니다.

그 결과, `terraform plan` 이나 `apply` 단계에서 수많은 예기치 않은 변경 사항이 발생하거나, 호환성 문제로 인해 오류가 발생할 수 있습니다.

**특정 버전을 명시적으로 고정하면(`version = "2.1.0"`), 이러한 위험을 원천적으로 차단할 수 있습니다.**

### 코드 예시: 모듈 버전 설정

```terraform
module "vpc" {
  # Terraform 레지스트리의 AWS VPC 모듈 사용
  source  = "terraform-aws-modules/vpc/aws"
  
  # 모듈 버전을 5.1.2로 '고정'합니다.
  # 이렇게 하면, 해당 모듈의 5.1.3이나 6.0.0 버전이 출시되어도
  # terraform init 시 항상 5.1.2 버전을 다운로드합니다.
  version = "5.1.2"

  # 모듈에 필요한 입력 변수들...
  name = "my-production-vpc"
  cidr = "10.0.0.0/16"
}

module "s3_bucket" {
  source  = "terraform-aws-modules/s3-bucket/aws"

  # 모듈 버전을 5.0.0 이상, 6.0.0 미만으로 제한합니다.
  # `terraform init -upgrade` 실행 시 이 범위 내의 최신 버전으로 업데이트됩니다.
  version = "~> 5.0"

  bucket = "my-unique-app-data-bucket"
}
```

## 다른 소스에서의 버전 관리

`version` 인수는 Terraform 레지스트리를 통해 배포되는 모듈에만 적용됩니다. 다른 소스를 사용할 경우, 해당 소스의 기능을 이용하여 버전을 고정해야 합니다.

*   **Git 저장소**: `source` URL에 `?ref=` 파라미터를 사용하여 특정 태그(tag), 브랜치(branch), 또는 커밋 해시(commit hash)를 명시하는 것이 가장 좋은 방법입니다.
    *   **권장**: 불변하는 태그(`?ref=v1.2.0`)나 커밋 해시(`?ref=abc1234`)를 사용합니다.
    *   **주의**: 브랜치(`?ref=main`)를 사용하면 해당 브랜치의 최신 커밋을 따라가므로, 의도치 않은 변경이 발생할 수 있습니다.

    ```terraform
    module "iam_role" {
      source = "git::https://github.com/example-org/terraform-aws-iam.git?ref=v3.5.1"
    }
    ```

*   **HTTP URL**: URL 경로 자체에 버전 번호를 포함시키는 것이 일반적인 패턴입니다.

    ```terraform
    module "database" {
      source = "https://my-artifact-repo.example.com/modules/rds/v1.2.0.zip"
    }
    ```

**로컬 경로 모듈은 `version` 인수를 지원하지 않습니다.** 로컬 모듈은 루트 구성과 함께 동일한 저장소에서 버전이 관리된다고 가정합니다.

---

## 예상 문제

1.  **Terraform 레지스트리에서 가져온 모듈의 버전을 명시적으로 지정하기 위해 `module` 블록에서 사용하는 인수는 무엇입니까?**<br>
    a. `tag`<br>
    b. `ref`<br>
    c. `module_version`<br>
    d. `version`<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>d</p>
    </details><br>

2.  **모듈의 `version`을 특정 버전으로 고정했을 때의 가장 큰 이점은 무엇입니까?**<br>
    a. `terraform init` 속도가 빨라진다.<br>
    b. 모듈 소스 코드가 변경되더라도 현재 구성에 영향을 주지 않아 예측 가능성과 안정성이 높아진다.<br>
    c. 모듈 사용 비용이 저렴해진다.<br>
    d. 모듈의 모든 출력을 자동으로 사용할 수 있다.<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>b</p>
    </details><br>

3.  **다음 중 모듈의 `version` 인수에 대한 설명으로 올바른 것은 무엇입니까?**<br>
    a. 모든 종류의 모듈 소스(`local`, `git` 등)에서 사용할 수 있다.<br>
    b. 프로바이더 버전 제약 조건과 동일한 구문을 사용한다.<br>
    c. 오직 정확한 버전(예: `= "1.2.3"`)만 지정할 수 있다.<br>
    d. `terraform.tfvars` 파일에서만 설정할 수 있다.<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>b</p>
    </details><br>

4.  **`source`가 로컬 경로(예: `./modules/vpc`)인 모듈에 `version` 인수를 사용하면 어떻게 됩니까?**<br>
    a. 로컬 모듈의 버전이 성공적으로 고정된다.<br>
    b. 경고 메시지가 출력되지만 정상적으로 동작한다.<br>
    c. 오류가 발생한다. `version` 인수는 로컬 경로 모듈에 사용할 수 없다.<br>
    d. `.git` 디렉토리의 태그를 읽어 버전을 확인한다.<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>c</p>
    </details><br>

5.  **Git 저장소를 모듈 소스로 사용할 때, 특정 커밋 해시(commit hash)를 참조하도록 버전을 고정하는 방법은 무엇입니까?**<br>
    a. `version = "<COMMIT_HASH>"`<br>
    b. `source = "git::https://...git#<COMMIT_HASH>"`<br>
    c. `source = "git::https://...git?ref=<COMMIT_HASH>"`<br>
    d. Git 모듈은 커밋 해시로 버전을 고정할 수 없다.<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>c</p>
    </details><br>

6.  **`version = "~> 3.1"` 제약 조건의 의미는 무엇입니까?**<br>
    a. 정확히 3.1.0 버전<br>
    b. 3.1.0 이상, 3.2.0 미만 (>= 3.1.0, < 3.2.0)<br>
    c. 3.1.0 이상, 4.0.0 미만 (>= 3.1.0, < 4.0.0)<br>
    d. 3.1.0을 포함한 모든 3.x 버전<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>c. `~>` 연산자는 마지막으로 명시된 버전 구성요소의 다음 단계를 올리는 것을 허용합니다. `~> 3.1`은 3.1.x 버전은 허용하지만 3.2.0은 허용하지 않고, 4.0.0도 허용하지 않습니다.</p>
    </details><br>

7.  **모듈의 버전을 업그레이드하기 위한 올바른 절차는 무엇입니까?**<br>
    a. `module` 블록의 `version` 제약 조건을 수정한 후, `terraform init -upgrade`를 실행한다.<br>
    b. `.terraform/modules` 디렉토리의 내용을 수동으로 변경한다.<br>
    c. `terraform module upgrade` 명령을 실행한다.<br>
    d. `terraform apply -upgrade` 명령을 실행한다.<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>a</p>
    </details><br>

8.  **안정적인 프로덕션(운영) 환경에서 Git 저장소를 모듈 소스로 사용할 때 가장 권장되는 `ref` 값은 무엇입니까?**<br>
    a. 브랜치 이름 (예: `main`)<br>
    b. 와일드카드 (예: `v1.*`)<br>
    c. 불변하는 릴리즈 태그 (예: `v2.1.5`)<br>
    d. `latest`<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>c. 브랜치는 내용이 계속 변경될 수 있으므로, 특정 시점의 코드를 보장하는 태그나 커밋 해시를 사용하는 것이 안전합니다.</p>
    </details><br>

9.  **Terraform은 모듈 버전에 대한 잠금 정보를 어디에 기록합니까?**<br>
    a. `terraform.tfstate` 파일<br>
    b. `.terraform.lock.hcl` 파일<br>
    c. `modules.tf` 파일<br>
    d. 별도로 기록하지 않는다.<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>b. 프로바이더와 마찬가지로, `terraform init` 시 선택된 모듈의 정보(레지스트리 모듈의 경우)가 의존성 잠금 파일에 기록되어 일관성을 보장합니다.</p>
    </details><br>

10. **`version` 인수를 생략하고 Terraform 레지스트리의 모듈을 사용하면 어떻게 됩니까?**<br>
    a. `terraform init` 실행 시 오류가 발생한다.<br>
    b. `terraform init`을 실행할 때마다 해당 모듈의 가장 최신 버전을 다운로드한다.<br>
    c. 항상 1.0.0 버전을 사용한다.<br>
    d. `required_providers`에 정의된 버전을 따라간다.<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>b. 이는 예기치 않은 변경을 유발할 수 있으므로 권장되지 않습니다.</p>
    </details><br>

---

[이전 학습](./5c-Describe-variable-scope-within-modules-child-modules.md) | [챕터 목록](./README.md) | [전체 목록](../../README.md) | [다음 학습](../6-Use-the-core-Terraform-workflow/README.md)
