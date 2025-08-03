[이전 학습](./4b-Use-terraform-state-to-view-Terraform-state.md) | [챕터 목록](./README.md) | [전체 목록](../README.md) | [다음 학습](../5-Interact-with-Terraform-modules/5a-Contrast-and-use-different-module-source-options-including-the-public-Terraform-Registry.md)

---

# 4c. 상세 로깅(Verbose Logging) 활성화 시점과 그 가치 설명

Terraform은 기본적으로 사용자가 꼭 알아야 할 정보만 간결하게 출력합니다. 하지만 때로는 Terraform이 내부적으로 어떻게 동작하는지, 왜 특정 작업이 실패하는지, 또는 프로바이더가 API와 어떻게 통신하는지 깊이 있게 들여다봐야 할 때가 있습니다.

이럴 때 사용하는 것이 바로 <b>상세 로깅(Verbose Logging)</b>입니다. 환경 변수를 통해 로깅 레벨을 조절하여 Terraform의 내부 동작에 대한 상세한 로그를 얻을 수 있습니다.

## 상세 로깅은 언제 활성화하는가?

상세 로깅은 주로 다음과 같은 문제 해결(Troubleshooting) 및 디버깅 시나리오에서 사용됩니다.

1.  **Terraform 내부 오류 디버깅**: `terraform plan`이나 `apply`가 알 수 없는 이유로 실패하거나 충돌(crash)할 때, 상세 로그를 통해 어떤 내부 연산에서 문제가 발생했는지 추적할 수 있습니다.
2.  **프로바이더 동작 확인**: 프로바이더가 클라우드 API와 주고받는 실제 요청(Request)과 응답(Response) 내용을 확인하고 싶을 때 사용합니다. API 요청이 올바르게 구성되었는지, API 응답에 예상치 못한 오류가 포함되어 있는지 등을 파악할 수 있습니다.
3.  **성능 문제 분석**: Terraform 작업이 비정상적으로 느릴 때, 로그를 분석하여 어떤 작업(예: 특정 리소스 조회)에서 시간이 오래 걸리는지 병목 현상을 파악할 수 있습니다.
4.  **복잡한 버그 리포팅**: Terraform이나 특정 프로바이더에 버그가 의심될 때, 상세 로그를 첨부하여 개발자에게 전달하면 문제 재현 및 원인 파악에 큰 도움이 됩니다.

## 상세 로깅 활성화 방법

Terraform의 로깅은 두 개의 환경 변수를 통해 제어됩니다.

1.  **`TF_LOG`**: 로그의 상세 수준(Level)을 설정합니다.
2.  **`TF_LOG_PATH`**: 로그를 저장할 파일 경로를 지정합니다. 지정하지 않으면 로그는 터미널(stderr)에 직접 출력됩니다.

### `TF_LOG` 레벨

가장 낮은 레벨(`TRACE`)이 가장 상세한 정보를 담고 있으며, 위로 갈수록 로그 양이 줄어듭니다.

| 레벨 | 설명 |
|---|---|
| `TRACE` | 가장 상세한 레벨. Terraform Core와 프로바이더의 모든 내부 동작, 통신 내용을 출력합니다. (가장 많이 사용) |
| `DEBUG` | 디버깅에 유용한 정보를 출력합니다. `TRACE`보다 덜 상세하지만 여전히 많은 정보를 제공합니다. |
| `INFO` | 일반적인 정보성 메시지를 출력합니다. |
| `WARN` | 경고 메시지를 출력합니다. |
| `ERROR` | 오류 메시지만 출력합니다. (기본 동작과 유사) |

### 사용 예시

**1. 터미널에 `DEBUG` 레벨 로그 출력하기**

```bash
export TF_LOG=DEBUG
terraform apply
```

**2. 파일에 `TRACE` 레벨 로그 저장하기**

로그가 매우 길어질 수 있으므로, 파일로 저장하여 분석하는 것이 일반적입니다.

```bash
export TF_LOG=TRACE
export TF_LOG_PATH="terraform.log"
terraform apply

# 작업이 끝나면 로그 파일(terraform.log)을 열어서 분석합니다.
```

로그 출력을 중단하려면 환경 변수를 해제하면 됩니다.

```bash
unset TF_LOG
unset TF_LOG_PATH
```

## 로그 결과의 가치 (Outcome and Value)

상세 로깅을 활성화하면 방대한 양의 텍스트가 출력되지만, 그 안에는 문제 해결의 실마리가 되는 귀중한 정보들이 포함되어 있습니다.

*   **명확한 실행 흐름**: Terraform이 어떤 순서로 작업을 처리하고 의존성을 계산하는지 명확히 볼 수 있습니다.
*   **API 상호작용 투명성**: 프로바이더가 생성하는 실제 API 요청의 헤더와 본문, 그리고 클라우드로부터 받는 응답을 그대로 볼 수 있어, 인증 문제나 권한 오류(예: "403 Forbidden")의 원인을 정확히 파악하는 데 결정적입니다.
*   **숨겨진 오류 발견**: 간결한 일반 출력에서는 보이지 않던 저수준의 오류 메시지나 경고를 발견할 수 있습니다.
*   **객관적인 증거**: 로그는 추측이 아닌 사실에 기반하여 문제를 분석할 수 있는 객관적인 데이터를 제공합니다.

비록 로그가 복잡해 보일 수 있지만, "error", "failed", "403", "500"과 같은 키워드로 검색하면 문제의 원인이 되는 부분을 빠르게 찾아낼 수 있습니다.

---

## 예상 문제

1.  **Terraform에서 상세 로깅(Verbose Logging)을 활성화하는 주된 목적은 무엇입니까?**<br>
    a. `terraform apply`의 실행 속도를 높이기 위해<br>
    b. Terraform 코드의 가독성을 높이기 위해<br>
    c. Terraform의 내부 동작을 디버깅하고 문제를 해결하기 위해<br>
    d. 상태 파일을 자동으로 백업하기 위해<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>c</p>
    </details><br>

2.  **Terraform의 로그 상세 수준을 설정하는 데 사용되는 환경 변수는 무엇입니까?**<br>
    a. `TF_LOG_LEVEL`<br>
    b. `TERRAFORM_LOG`<br>
    c. `TF_LOG`<br>
    d. `TF_DEBUG`<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>c</p>
    </details><br>

3.  **가장 상세한 수준의 로그를 얻기 위해 `TF_LOG` 환경 변수에 설정해야 하는 값은 무엇입니까?**<br>
    a. `DEBUG`<br>
    b. `INFO`<br>
    c. `DETAIL`<br>
    d. `TRACE`<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>d</p>
    </details><br>

4.  **Terraform 로그를 터미널이 아닌 특정 파일(예: `tf.log`)에 저장하고 싶을 때 설정해야 하는 환경 변수는 무엇입니까?**<br>
    a. `TF_LOG_FILE`<br>
    b. `TF_LOG_PATH`<br>
    c. `TF_OUTPUT_PATH`<br>
    d. `TERRAFORM_LOG_FILE`<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>b</p>
    </details><br>

5.  **다음 중 상세 로깅이 가장 유용한 시나리오는 무엇입니까?**<br>
    a. 새로운 팀원에게 Terraform 기본 워크플로우를 교육할 때<br>
    b. `terraform plan` 실행 시 프로바이더가 알 수 없는 500 서버 오류를 반환하며 실패할 때<br>
    c. 매일 정기적으로 `terraform apply`를 실행할 때<br>
    d. Terraform 구성 파일의 변수 값을 확인하고 싶을 때<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>b. 프로바이더와 API 간의 통신 문제를 디버깅하는 것은 상세 로깅의 핵심 사용 사례입니다.</p>
    </details><br>

6.  **`export TF_LOG=INFO` 명령을 실행한 후 `terraform apply`를 실행했습니다. 어떤 종류의 로그를 볼 수 있습니까?**<br>
    a. `INFO`, `WARN`, `ERROR` 레벨의 로그<br>
    b. 오직 `INFO` 레벨의 로그<br>
    c. `TRACE`, `DEBUG`, `INFO`, `WARN`, `ERROR` 모든 레벨의 로그<br>
    d. 아무 로그도 볼 수 없다.<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>a. 로그 레벨을 설정하면 해당 레벨과 그보다 심각한(상위) 레벨의 로그가 모두 출력됩니다.</p>
    </details><br>

7.  **상세 로깅의 결과물(outcome)에서 얻을 수 있는 가치(value)가 아닌 것은 무엇입니까?**<br>
    a. 프로바이더가 API와 주고받는 실제 요청 및 응답 확인<br>
    b. Terraform Core 내부의 구체적인 오류 메시지 확인<br>
    c. Terraform 구성 파일(.tf)의 문법 오류 자동 수정<br>
    d. 성능 병목 현상 분석을 위한 단서 제공<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>c. 로깅은 문제 진단을 위한 정보를 제공할 뿐, 코드를 직접 수정하지는 않습니다.</p>
    </details><br>

8.  **Terraform 로그 파일에 민감한 정보(예: API 키, 비밀번호)가 포함될 수 있습니까?**<br>
    a. 아니요, Terraform은 모든 민감한 정보를 자동으로 마스킹 처리합니다.<br>
    b. 예, 로그에는 프로바이더가 API에 보내는 요청 전문이 포함될 수 있으므로 민감한 값이 노출될 수 있습니다. 로그 파일을 신중하게 다뤄야 합니다.<br>
    c. `TRACE` 레벨에서만 포함되고 `DEBUG` 레벨에서는 제외됩니다.<br>
    d. 로그 파일이 자동으로 암호화되므로 안전합니다.<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>b</p>
    </details><br>

9.  **로깅을 활성화한 후, 비활성화하려면 어떻게 해야 합니까?**<br>
    a. `TF_LOG=NONE` 으로 설정한다.<br>
    b. `unset TF_LOG` 명령으로 환경 변수를 제거한다.<br>
    c. Terraform을 재시작한다.<br>
    d. `terraform logging off` 명령을 실행한다.<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>b</p>
    </details><br>

10. **`TF_LOG_PATH`를 설정하지 않고 `TF_LOG=DEBUG`만 설정하면 로그는 어디에 출력됩니까?**<br>
    a. 현재 디렉토리의 `terraform.log` 파일<br>
    b. 터미널의 표준 출력(stdout)<br>
    c. 터미널의 표준 에러(stderr)<br>
    d. 로그가 생성되지 않는다.<br>
    <br>
    <details>
    <summary>정답 확인</summary>
    <p>c</p>
    </details><br>

---

[이전 학습](./4b-Use-terraform-state-to-view-Terraform-state.md) | [챕터 목록](./README.md) | [전체 목록](../../README.md) | [다음 학습](../5-Interact-with-Terraform-modules/README.md)
