[이전 학습](./3b-Describe-plugin-based-architecture.md) | [챕터 목록](./README.md) | [전체 목록](../README.md) | [다음 학습](./3d-Describe-how-Terraform-finds-and-fetches-providers.md)

---

# 3c. 여러 프로바이더를 사용하여 Terraform 구성 작성하기

Terraform의 강력한 기능 중 하나는 단일 구성 내에서 여러 클라우드 제공업체나 서비스를 동시에 관리할 수 있다는 것입니다. 예를 들어, AWS에 서버를 배포하면서 동시에 GCP의 DNS 레코드를 업데이트하거나, Datadog에 모니터링을 설정하는 등의 작업을 하나의 `terraform apply` 명령으로 처리할 수 있습니다.

## 여러 프로바이더 구성 방법

여러 프로바이더를 사용하는 방법은 간단합니다. `terraform` 블록의 `required_providers` 안에 사용할 모든 프로바이더를 정의하고, 각 프로바이더에 대한 `provider` 구성 블록을 선언하면 됩니다.

### 기본 프로바이더와 별칭(Alias) 프로바이더

Terraform은 기본적으로 `provider` 블록의 이름(예: "aws", "google")을 해당 타입의 **기본(Default)** 프로바이더 구성으로 인식합니다.

하지만 같은 종류의 프로바이더를 여러 개 사용해야 할 때가 있습니다. 예를 들어, 서로 다른 AWS 리전(region)이나 다른 AWS 계정에 리소스를 동시에 배포해야 하는 경우입니다. 이때 **프로바이더 별칭(alias)** 기능을 사용합니다.

`provider` 블록에 `alias` 메타-인수를 추가하여 프로바이더 구성에 별명을 붙일 수 있습니다. 그리고 이 별칭은 리소스나 모듈을 특정 프로바이더 구성과 연결할 때 `provider` 메타-인수를 통해 사용됩니다.

## Terraform 코드 예시: AWS의 두 리전에 인스턴스 배포하기

다음 예시는 AWS 프로바이더를 두 번 구성하여, 하나는 `us-east-1` 리전(기본)에, 다른 하나는 `ap-northeast-2` 리전(별칭 "seoul")에 EC2 인스턴스를 각각 배포하는 방법을 보여줍니다.

```terraform
# 1. 사용할 프로바이더 정의
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

# 2. 기본(Default) 프로바이더 구성: us-east-1 리전
provider "aws" {
  region = "us-east-1"
}

# 3. 별칭(Alias) 프로바이더 구성: ap-northeast-2 리전
provider "aws" {
  alias  = "seoul"
  region = "ap-northeast-2"
}

# 4. 기본 프로바이더(us-east-1)를 사용하여 리소스 생성
# provider 메타-인수가 없으면 기본 프로바이더를 사용합니다.
resource "aws_instance" "web_us" {
  ami           = "ami-0c55b159cbfafe1f0" # us-east-1의 AMI
  instance_type = "t2.micro"

  tags = {
    Name = "WebServer-US-East-1"
  }
}

# 5. 별칭 프로바이더(ap-northeast-2)를 사용하여 리소스 생성
# provider 메타-인수에 "aws.<별칭>" 형식으로 지정합니다.
resource "aws_instance" "web_kr" {
  provider = aws.seoul

  ami           = "ami-0c93a3c253c5a31f5" # ap-northeast-2의 AMI
  instance_type = "t2.micro"

  tags = {
    Name = "WebServer-AP-Northeast-2"
  }
}
```

### 코드 설명

*   **기본 프로바이더 사용**: `resource "aws_instance" "web_us"` 블록에는 `provider` 메타-인수가 없습니다. 이 경우 Terraform은 `aws` 타입의 기본 프로바이더 구성(즉, `us-east-1` 리전을 사용하는 구성)을 자동으로 사용합니다.
*   **별칭 프로바이더 사용**: `resource "aws_instance" "web_kr"` 블록에는 `provider = aws.seoul` 이라는 명시적인 지시가 있습니다. Terraform은 이 지시에 따라 `alias`가 "seoul"로 설정된 `aws` 프로바이더 구성(즉, `ap-northeast-2` 리전을 사용하는 구성)을 사용하여 리소스를 생성합니다.

이러한 방식으로 사용자는 단일 Terraform 프로젝트 내에서 여러 클라우드, 여러 리전, 여러 계정에 걸친 복잡한 인프라를 일관되고 체계적으로 관리할 수 있습니다.

---

## 예상 문제

1.  **하나의 Terraform 구성에서 서로 다른 두 개의 AWS 리전에 리소스를 배포하기 위해 사용해야 하는 기능은 무엇입니까?**<br>
    a. `terraform workspace`<br>
    b. `for_each` 메타-인수<br>
    c. 프로바이더 별칭 (Provider Alias)<br>
    d. `terraform import`<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>c</p>
    </details><br>

2.  **`provider` 블록에서 별칭(alias)을 정의하는 데 사용되는 메타-인수는 무엇입니까?**<br>
    a. `name`<br>
    b. `label`<br>
    c. `id`<br>
    d. `alias`<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>d</p>
    </details><br>

3.  **리소스 블록에서 특정 별칭 프로바이더를 사용하도록 지정하려면 어떻게 해야 합니까? (프로바이더 타입: `aws`, 별칭: `staging`)**<br>
    a. `provider = "staging"`<br>
    b. `provider = aws.staging`<br>
    c. `alias = "staging"`<br>
    d. `provider_alias = "aws.staging"`<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>b</p>
    </details><br>

4.  **리소스 블록에 `provider` 메타-인수가 명시적으로 지정되지 않은 경우, Terraform은 어떤 프로바이더 구성을 사용합니까?**<br>
    a. 알파벳 순으로 첫 번째 프로바이더 구성<br>
    b. 해당 리소스 타입의 기본(Default) 프로바이더 구성<br>
    c. `required_providers`에 가장 먼저 정의된 프로바이더<br>
    d. 오류를 발생시키며 중단됨<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>b</p>
    </details><br>

5.  **다음 중 여러 프로바이더를 사용하는 적절한 사례가 아닌 것은 무엇입니까?**<br>
    a. AWS에 EC2 인스턴스를 만들고 Cloudflare에서 DNS 레코드를 관리<br>
    b. 서로 다른 두 개의 GCP 프로젝트에 리소스를 배포<br>
    c. 개발 환경과 운영 환경을 분리하기 위해 `terraform workspace`를 사용<br>
    d. AWS 미국 동부 리전과 유럽 서부 리전에 재해 복구(DR) 환경을 구축<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>c. `workspace`는 동일한 구성으로 여러 상태를 관리하는 기능이며, 여러 프로바이더를 사용하는 것과는 다른 목적의 기능입니다.</p>
    </details><br>

6.  **여러 프로바이더를 사용하면 `terraform.tfstate` 파일은 어떻게 됩니까?**<br>
    a. 프로바이더 개수만큼 여러 개의 상태 파일이 생성된다.<br>
    b. 단일 상태 파일 내에 모든 프로바이더가 관리하는 리소스 정보가 함께 저장된다.<br>
    c. 상태 파일이 더 이상 사용되지 않는다.<br>
    d. 상태 파일 이름이 `terraform.tfstate.multi`로 변경된다.<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>b</p>
    </details><br>

7.  **기본 프로바이더 구성(Default Provider Configuration)이란 무엇을 의미합니까?**<br>
    a. `alias` 메타-인수가 없는 `provider` 블록<br>
    b. `required_providers` 블록에 정의된 첫 번째 프로바이더<br>
    c. HashiCorp가 권장하는 프로바이더 설정<br>
    d. `provider` 블록이 없는 경우의 기본 설정<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>a</p>
    </details><br>

8.  **AWS 프로바이더와 GCP 프로바이더를 동시에 사용하고 있습니다. `terraform plan` 명령을 실행하면 어떤 일이 발생합니까?**<br>
    a. AWS에 대한 계획만 생성되고 GCP는 무시된다.<br>
    b. 오류가 발생한다. 한 번에 하나의 프로바이더만 사용할 수 있다.<br>
    c. Terraform은 두 프로바이더 모두에 대해 API를 호출하여 현재 상태를 확인하고, 변경 계획을 종합하여 출력한다.<br>
    d. 사용자가 대화형 프롬프트에서 어떤 프로바이더를 계획할지 선택해야 한다.<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>c</p>
    </details><br>

9.  **`provider "aws"` 블록 안에 `alias = "default"` 라고 명시하는 것은 유효합니까?**<br>
    a. 예, 기본 프로바이더임을 명확히 하기 위해 사용할 수 있습니다.<br>
    b. 아니요, "default"는 예약어이므로 별칭으로 사용할 수 없습니다.<br>
    c. 예, 하지만 아무런 효과가 없습니다.<br>
    d. 아니요, 별칭은 숫자로만 지정해야 합니다.<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>b. `default`는 키워드이므로 별칭으로 사용하면 충돌이 발생할 수 있습니다.</p>
    </details><br>

10. **모듈(module)에 특정 프로바이더 구성을 전달하려면 어떻게 해야 합니까?**<br>
    a. 모듈 블록 내에 `provider` 블록을 직접 정의한다.<br>
    b. 모듈 블록의 `providers` 메타-인수를 사용한다.<br>
    c. 모듈은 항상 기본 프로바이더만 사용하므로 전달할 수 없다.<br>
    d. 전역 변수를 통해 프로바이더 설정을 전달한다.<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>b. 모듈에 프로바이더 구성을 전달할 때는 `providers` 맵을 사용합니다. 예: `providers = { aws = aws.seoul }`</p>
    </details><br>

---

[이전 학습](./3b-Describe-plugin-based-architecture.md) | [챕터 목록](./README.md) | [전체 목록](../../README.md) | [다음 학습](./3d-Describe-how-Terraform-finds-and-fetches-providers.md)
