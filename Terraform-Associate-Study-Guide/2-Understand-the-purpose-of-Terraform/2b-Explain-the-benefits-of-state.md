[이전 학습](./2a-Explain-multi-cloud-and-provider-agnostic-benefits.md) | [챕터 목록](./README.md) | [전체 목록](../README.md) | [다음 학습](../3-Understand-Terraform-basics/3a-Install-and-version-Terraform-providers.md)

---

# 2b. Terraform 상태(State)의 이점 설명

## Terraform 상태(State)란 무엇인가?

Terraform 상태는 Terraform이 관리하는 인프라 리소스에 대한 정보를 저장하는 파일(기본적으로 `terraform.tfstate`)입니다. 이 파일은 Terraform의 "두뇌"와 같아서, 구성 파일(`.tf`)과 실제 클라우드 환경에 존재하는 리소스 사이의 연결고리 역할을 합니다.

상태 파일이 없다면, Terraform은 자신이 어떤 리소스를 생성했는지, 그리고 그 리소스들이 현재 어떤 상태인지 전혀 알 수 없게 됩니다.

## 상태(State)의 주요 이점

1.  **리소스 매핑 (Mapping to the Real World)**
    *   상태 파일의 가장 핵심적인 기능은 Terraform 구성 파일에 정의된 리소스(예: `resource "aws_instance" "web"`)와 실제 클라우드에 생성된 리소스(예: EC2 인스턴스 ID `i-1234567890abcdef0`)를 1:1로 매핑하는 것입니다. 이 매핑 정보 덕분에 Terraform은 어떤 리소스를 변경하거나 삭제해야 할지 정확히 알 수 있습니다.

2.  **성능 향상 (Performance)**
    *   만약 상태 파일이 없다면, Terraform은 변경 사항을 계산하기 위해 모든 클라우드 리소스를 일일이 조회해야 합니다. 대규모 인프라에서는 수많은 API 호출이 발생하여 매우 느리고 비효율적일 것입니다. 상태 파일은 필요한 리소스 정보의 로컬 캐시 역할을 하여, `plan`과 `apply` 작업의 성능을 크게 향상시킵니다.

3.  **의존성 관리 (Dependency Tracking)**
    *   Terraform은 리소스 간의 의존성을 자동으로 파악하고 상태 파일에 기록합니다. 예를 들어, EC2 인스턴스를 생성하려면 먼저 VPC와 서브넷이 있어야 합니다. Terraform은 이 의존성 순서를 상태 파일을 통해 인지하고, 리소스를 올바른 순서로 생성하거나 삭제합니다. 이를 통해 복잡한 인프라도 안전하게 관리할 수 있습니다.

4.  **팀 협업 (Collaboration)**
    *   상태 파일을 로컬 컴퓨터가 아닌 원격 백엔드(Remote Backend, 예: S3, Terraform Cloud)에 저장하면, 여러 팀원이 동일한 인프라에 대해 안전하게 협업할 수 있습니다. 원격 백엔드는 상태 파일에 대한 잠금(Locking) 기능을 제공하여, 두 명 이상의 사용자가 동시에 `apply`를 실행하여 상태 파일이 깨지는 것을 방지합니다.

## Terraform 상태 관련 명령어 예시

Terraform은 상태 파일을 직접 조작할 수 있는 `terraform state` 명령어를 제공합니다. 이 명령어는 매우 강력하므로 주의해서 사용해야 합니다.

```bash
# 현재 상태 파일에 기록된 모든 리소스 목록 보기
terraform state list

# 특정 리소스(예: aws_instance.web)의 상세 정보 보기
terraform state show aws_instance.web

# Terraform 구성 파일에는 없지만, 상태 파일에 남아있는 리소스를 상태 파일에서 제거
# (실제 인프라는 삭제되지 않음)
terraform state rm aws_instance.old_server

# 로컬 상태를 원격 백엔드의 최신 상태로 업데이트
terraform refresh
```

이처럼 상태 파일은 Terraform이 인프라를 지능적이고 안전하게 관리하기 위한 필수적인 요소입니다.

---

## 예상 문제

1.  **Terraform 상태 파일(state file)의 가장 중요한 역할은 무엇입니까?**<br>
    a. Terraform 코드의 백업을 저장하는 것<br>
    b. 구성 파일의 리소스와 실제 세계의 리소스를 매핑하는 것<br>
    c. 클라우드 제공업체의 API 자격 증명을 저장하는 것<br>
    d. `terraform plan`의 실행 결과를 저장하는 것<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>b</p>
    </details><br>

2.  **상태 파일이 없으면 `terraform plan` 실행 시 어떤 일이 발생할 가능성이 높습니까?**<br>
    a. 항상 아무 변경 사항이 없다고 출력된다.<br>
    b. 모든 리소스를 새로 생성하려고 시도한다.<br>
    c. 오류를 발생시키며 즉시 중단된다.<br>
    d. 원격지의 모든 리소스를 조회하여 성능이 크게 저하된다.<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>b</p>
    </details><br>

3.  **Terraform이 리소스 생성 및 삭제 순서를 결정하는 데 사용하는 정보는 어디에 저장됩니까?**<br>
    a. `.tf` 구성 파일<br>
    b. `.tfvars` 변수 파일<br>
    c. `.tfstate` 상태 파일<br>
    d. 프로바이더 플러그인 내부<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>c</p>
    </details><br>

4.  **팀 협업 시 상태 파일을 로컬 머신이 아닌 원격 백엔드(Remote Backend)에 저장하는 주된 이유는 무엇입니까?**<br>
    a. 상태 파일의 크기를 줄이기 위해<br>
    b. 여러 사용자가 동시에 상태를 수정하는 것을 방지하는 잠금(Locking) 기능을 사용하기 위해<br>
    c. `terraform plan` 실행 속도를 높이기 위해<br>
    d. 상태 파일을 시각적으로 보기 위해<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>b</p>
    </details><br>

5.  **`terraform state rm` 명령어는 어떤 작업을 수행합니까?**<br>
    a. 실제 클라우드 인프라와 상태 파일을 모두 삭제한다.<br>
    b. 실제 인프라는 그대로 두고, 상태 파일에서만 특정 리소스에 대한 추적을 중단한다.<br>
    c. 상태 파일을 백업한다.<br>
    d. 상태 파일의 이름을 변경한다.<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>b</p>
    </details><br>

6.  **Terraform 외부에서(예: 클라우드 콘솔에서 직접) 리소스가 변경되었을 때, 이 변경 사항을 상태 파일에 반영하기 위해 사용할 수 있는 명령어는 무엇입니까?**<br>
    a. `terraform apply`\
    b. `terraform state push`\
    c. `terraform refresh`\
    d. `terraform import`<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>c</p>
    </details><br>

7.  **기본적으로 Terraform 상태 파일의 이름은 무엇입니까?**<br>
    a. `terraform.tfvars`\
    b. `state.json`\
    c. `terraform.tfstate`\
    d. `main.tf`<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>c</p>
    </details><br>

8.  **상태 파일에 민감한 정보(예: 데이터베이스 비밀번호)가 일반 텍스트로 저장될 수 있다는 보안 문제를 해결하기 위한 가장 좋은 방법은 무엇입니까?**<br>
    a. 상태 파일을 Git에 커밋하지 않는다.<br>
    b. 상태 파일을 암호화하는 원격 백엔드를 사용한다.<br>
    c. 상태 파일에서 민감한 부분을 수동으로 삭제한다.<br>
    d. 변수 파일에 민감한 정보를 저장한다.<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>b</p>
    </details><br>

9.  **`terraform state list` 명령어의 용도는 무엇입니까?**<br>
    a. 사용 가능한 모든 Terraform 명령어를 나열한다.<br>
    b. 현재 상태 파일이 추적하고 있는 모든 리소스의 목록을 보여준다.<br>
    c. 원격 백엔드의 모든 상태 파일 목록을 보여준다.<br>
    d. 현재 디렉토리의 모든 `.tf` 파일 목록을 보여준다.<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>b</p>
    </details><br>

10. **Terraform이 `plan`을 생성할 때, 무엇과 무엇을 비교합니까?**<br>
    a. 이전 상태 파일과 현재 상태 파일<br>
    b. 현재 구성과 상태 파일에 기록된 이전 상태<br>
    c. 원격 인프라와 로컬 인프라<br>
    d. `main.tf` 파일과 `variables.tf` 파일<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>b</p>
    </details><br>

---

[이전 학습](./2a-Explain-multi-cloud-and-provider-agnostic-benefits.md) | [챕터 목록](./README.md) | [전체 목록](../../README.md) | [다음 학습](../../3-Understand-Terraform-basics/README.md)
