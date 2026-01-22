<img width="568" height="246" alt="image" src="https://github.com/user-attachments/assets/e6ffdc31-2ef5-4d10-9657-c01616731aaf" />

<br>

> **Authentication(인증)과 Authorization(인가)는** 보안 시스템에서 서로 다른 역할을 가진 개념이다.

- **인증**: 사용자가 누구인지 확인 → 신원 확인 단계
- **인가**: 사용자가 무엇을 할 수 있는지 결정 → 권한 판단 단계

<br>

### Authentication (인증)

> Authentication은 사용자의 신원을 검증하는 과정이다.

- **목적:** 사용자 식별
- **결과:** “이 요청은 누구의 요청인가”
- **특징:** 성공 시 사용자 식별 정보가 시스템에 전달됨

<br>

### Authorization (인가)

> Authorization은 인증된 사용자가 특정 자원에 접근할 권한이 있는지 판단하는 과정이다.

- **목적:** 접근 제어
- **기준:** Role, Permission, Scope 등
- **전제:** 인증이 완료된 사용자만 대상이 됨

<br>
