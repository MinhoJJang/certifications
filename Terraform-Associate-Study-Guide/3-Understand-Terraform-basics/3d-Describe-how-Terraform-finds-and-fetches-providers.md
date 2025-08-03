[이전 학습](./3c-Write-Terraform-configuration-using-multiple-providers.md) | [챕터 목록](./README.md) | [전체 목록](../README.md) | [다음 학습](../4-Use-Terraform-outside-the-core-workflow/4a-Describe-when-to-use-terraform-import-to-import-existing-infrastructure-into-your-Terraform-state.md)

---

# 3d. Terraform이 프로바이더를 찾고 가져오는 방법 설명

`terraform init`은 Terraform 워크플로우의 첫 번째 단계이며, 이 명령의 핵심 역할 중 하나는 구성에 필요한 모든 프로바이더를 찾아서 다운로드하는 것입니다. Terraform은 이 과정을 체계적으로 수행하기 위해 <b>프로바이더 레지스트리(Provider Registry)</b>를 사용합니다.

## 프로바이더 소스 주소 (Provider Source Addresses)

Terraform이 특정 프로바이더를 고유하게 식별하기 위해 사용하는 주소 형식입니다. 이 주소는 `required_providers` 블록의 `source` 속성에 명시됩니다.

**주소 형식**: `[<HOSTNAME>/]<NAMESPACE>/<TYPE>`

*   **`HOSTNAME` (선택 사항)**: 프로바이더가 호스팅되는 레지스트리의 도메인 이름입니다. 생략될 경우 기본값은 Terraform의 공식 레지스트리인 `registry.terraform.io` 입니다.
*   **`NAMESPACE`**: 프로바이더를 게시한 조직이나 사용자의 이름입니다. 예를 들어, HashiCorp가 공식적으로 지원하는 프로바이더는 `hashicorp` 네임스페이스를 사용합니다.
*   **`TYPE`**: 프로바이더의 이름입니다. 관리하려는 플랫폼의 이름(예: `aws`, `google`, `azuread`)이 주로 사용됩니다.

**예시**:
*   `hashicorp/aws`: 공식 Terraform 레지스트리(`registry.terraform.io`)에 있는, `hashicorp` 네임스페이스의 `aws` 프로바이더를 의미합니다.
*   `my-private-registry.com/my-org/internal-tool`: `my-private-registry.com`이라는 사설 레지스트리에 있는, `my-org` 네임스페이스의 `internal-tool` 프로바이더를 의미합니다.

## `terraform init`의 프로바이더 탐색 및 설치 과정

`terraform init`을 실행하면 Terraform Core는 다음 단계를 거쳐 프로바이더를 설치합니다.

1.  **구성 스캔**: 현재 작업 디렉토리의 `.tf` 파일들을 모두 스캔하여 `terraform { required_providers { ... } }` 블록을 찾습니다.
2.  **소스 주소 및 버전 제약 조건 확인**: `required_providers` 블록에 명시된 모든 프로바이더의 `source` 주소와 `version` 제약 조건을 확인합니다.
3.  **의존성 잠금 파일 확인 (`.terraform.lock.hcl`)**:
    *   **잠금 파일이 있고, 필요한 프로바이더 정보가 포함된 경우**: Terraform은 잠금 파일에 기록된 정확한 버전과 해시값을 사용하여 해당 프로바이더를 설치합니다. 이를 통해 팀 전체가 동일한 프로바이더 버전을 사용하도록 보장합니다. 프로바이더 레지스트리에 요청을 보내지 않을 수도 있습니다.
    *   **잠금 파일이 없거나, 새로운 프로바이더가 추가된 경우**: 다음 단계인 레지스트리 조회를 수행합니다.
4.  **프로바이더 레지스트리 조회**: `source` 주소에 명시된 레지스트리(기본값은 `registry.terraform.io`)에 API 요청을 보냅니다. 이 요청을 통해 `version` 제약 조건을 만족하는 사용 가능한 프로바이더 버전 목록과 각 버전의 다운로드 URL, 서명 키, 해시값(SHA)을 받습니다.
5.  **프로바이더 다운로드 및 검증**:
    *   조회된 정보 중 버전 제약 조건을 만족하는 최신 버전을 다운로드합니다.
    *   다운로드한 파일의 해시값을 계산하여 레지스트리가 제공한 해시값과 일치하는지 확인합니다. 이 과정을 통해 파일이 전송 중에 손상되지 않았는지, 악의적으로 변경되지 않았는지 검증합니다.
6.  **로컬에 설치**: 검증이 완료된 프로바이더 플러그인 파일을 현재 디렉토리의 `.terraform/providers/...` 경로 아래에 설치합니다.
7.  **잠금 파일 업데이트**: 새로 다운로드한 프로바이더의 최종 버전과 해시 정보를 `.terraform.lock.hcl` 파일에 기록하거나 업데이트합니다.

이러한 과정을 통해 Terraform은 사용자가 명시한 요구사항에 맞춰 안전하고 일관된 방식으로 프로바이더를 자동으로 관리합니다.

---

## 예상 문제

1.  **`terraform init` 명령의 주요 역할은 무엇입니까?**<br>
    a. 인프라 변경 사항을 미리 계산하여 보여준다.<br>
    b. 작업 디렉토리를 초기화하고, 백엔드를 설정하며, 필요한 프로바이더를 다운로드한다.<br>
    c. 현재 상태 파일과 실제 인프라의 차이점을 동기화한다.<br>
    d. 구성 파일의 문법 오류를 검사하고 수정한다.<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>b</p>
    </details><br>

2.  **`required_providers` 블록에 있는 `source` 속성의 역할은 무엇입니까?**<br>
    a. 프로바이더 코드의 원본 소스 코드 저장소 주소를 지정한다.<br>
    b. 프로바이더의 버전을 지정한다.<br>
    c. Terraform이 어떤 레지스트리에서 어떤 프로바이더를 찾아야 하는지 알려주는 고유 주소를 지정한다.<br>
    d. 프로바이더 개발자의 이름을 지정한다.<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>c</p>
    </details><br>

3.  **프로바이더 `source` 주소에서 `HOSTNAME` 부분을 생략하면 Terraform은 어디에서 프로바이더를 찾습니까?**<br>
    a. 로컬 파일 시스템<br>
    b. GitHub.com<br>
    c. Docker Hub<br>
    d. 공식 Terraform 레지스트리 (`registry.terraform.io`)<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>d</p>
    </details><br>

4.  **`terraform init`을 실행할 때, 이미 `.terraform.lock.hcl` 파일에 필요한 프로바이더 정보가 있다면 어떤 일이 발생합니까?**<br>
    a. 항상 레지스트리에 최신 버전이 있는지 확인하고 업그레이드한다.<br>
    b. 잠금 파일을 무시하고 `required_providers`의 버전 제약 조건에 맞는 최신 버전을 다운로드한다.<br>
    c. 잠금 파일에 명시된 정확한 버전을 다운로드하거나 확인하여 일관성을 유지한다.<br>
    d. 사용자에게 어떤 버전을 설치할지 물어본다.<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>c</p>
    </details><br>

5.  **프로바이더 플러그인은 다운로드 후 어디에 저장됩니까?**<br>
    a. `/usr/local/bin`<br>
    b. 사용자의 홈 디렉토리 (`~/.terraform.d/`)<br>
    c. 현재 작업 디렉토리의 `.terraform/providers/` 하위 경로<br>
    d. `/etc/terraform/providers`<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>c</p>
    </details><br>

6.  **Terraform이 프로바이더를 다운로드한 후 해시값을 검증하는 주된 이유는 무엇입니까?**<br>
    a. 다운로드 속도를 측정하기 위해<br>
    b. 프로바이더의 라이선스가 유효한지 확인하기 위해<br>
    c. 다운로드한 파일이 손상되거나 악의적으로 변경되지 않았음을 보장하기 위해 (보안 및 무결성)<br>
    d. 프로바이더가 현재 운영체제와 호환되는지 확인하기 위해<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>c</p>
    </details><br>

7.  **다음 중 유효한 프로바이더 `source` 주소가 아닌 것은 무엇입니까?**<br>
    a. `hashicorp/aws`<br>
    b. `my-registry.example.com/my-org/my-provider`<br>
    c. `gcp` (네임스페이스가 없음)<br>
    d. `integrations/github`<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>c. `NAMESPACE/TYPE` 형식은 필수입니다. `hashicorp/google`과 같이 네임스페이스가 포함되어야 합니다.</p>
    </details><br>

8.  **`terraform init` 과정에 대한 설명으로 틀린 것은 무엇입니까?**<br>
    a. `.tf` 파일을 스캔하여 필요한 프로바이더를 식별한다.<br>
    b. `.terraform.lock.hcl`이 있으면 해당 정보를 우선적으로 사용한다.<br>
    c. 프로바이더 플러그인을 다운로드하고 해시를 검증한다.<br>
    d. `terraform.tfstate` 파일을 생성하거나 업데이트한다.<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>d. `terraform init`은 상태 파일을 직접적으로 생성하거나 크게 수정하지 않습니다. 상태 파일은 `terraform apply` 이후에 주로 생성되고 내용이 채워집니다.</p>
    </details><br>

9.  **사내에서 개발한 커스텀 프로바이더를 사용하려면 어떻게 해야 합니까?**<br>
    a. Terraform 소스 코드를 직접 수정해야 한다.<br>
    b. 사설 프로바이더 레지스트리를 구축하거나, 파일 시스템 미러링 디렉토리를 설정하여 Terraform이 찾을 수 있도록 한다.<br>
    c. HashiCorp에 프로바이더 등록을 요청해야만 한다.<br>
    d. `.terraform/` 디렉토리에 수동으로 복사해 넣으면 항상 동작한다.<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>b. 사설 레지스트리를 구축하거나 로컬 디렉토리를 사용하는 것이 정식적인 방법입니다.</p>
    </details><br>

10. **`terraform init -upgrade` 옵션은 프로바이더 탐색 과정에 어떤 영향을 줍니까?**<br>
    a. `.terraform.lock.hcl` 파일을 무시하고, `required_providers` 블록의 버전 제약 조건 내에서 사용 가능한 최신 버전으로 업그레이드를 시도한다.<br>
    b. Terraform Core 바이너리 자체를 업그레이드한다.<br>
    c. 모든 프로바이더를 삭제하고 다시 다운로드한다.<br>
    d. 프로바이더 다운로드 과정을 생략한다.<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>a</p>
    </details><br>

---

[이전 학습](./3c-Write-Terraform-configuration-using-multiple-providers.md) | [챕터 목록](./README.md) | [전체 목록](../../README.md) | [다음 학습](../4-Use-Terraform-outside-the-core-workflow/README.md)
