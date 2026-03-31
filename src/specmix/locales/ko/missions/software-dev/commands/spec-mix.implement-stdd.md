---
description: STDD (Spec & Test Driven Development) 방법론으로 구현 실행
scripts:
  sh: scripts/bash/check-prerequisites.sh --json --require-tasks --include-tasks
  ps: scripts/powershell/check-prerequisites.ps1 -Json -RequireTasks -IncludeTasks
---

## 사용자 입력

```text
$ARGUMENTS
```

## 개요

STDD는 각 구현 단위(페이즈 또는 워크 패키지)를 5단계 사이클로 실행합니다.
Coder/Tester 역할을 분리하고 Error-as-Prompt 피드백 루프를 활용합니다.

**핵심 원칙:**

- **Zero-Ambiguity**: 모든 명세는 O/X(통과/실패) 기준, 주관적 표현 금지
- **Test-First**: 구현 전 테스트 먼저 작성
- **Decoupled Roles**: Coder와 Tester를 분리하여 할루시네이션 연쇄 차단
- **Error as Prompt**: 테스트 실패 로그가 곧 다음 지시

## Step 1: 설정

### 1.1 컨텍스트 로드

전제조건 스크립트를 실행하여 FEATURE_DIR을 가져옵니다:

```bash
{SCRIPT}
```

JSON 출력에서 `FEATURE_DIR`을 파싱합니다.

순서대로 읽기:

1. `$FEATURE_DIR/plan.md` 또는 `$FEATURE_DIR/tasks.md` (필수)
2. `specs/constitution.md` (존재 시)

### 1.2 STDD 디렉토리 생성

```bash
mkdir -p $FEATURE_DIR/stdd
```

### 1.3 에이전트 구성

```text
[STDD] 에이전트 구성:

| 옵션   | 설명                                                   |
|--------|--------------------------------------------------------|
| multi  | 4명의 에이전트: spec-analyst, coder, tester, loop-driver |
| single | 단일 에이전트, 역할 분리 수행 (기본값)                     |

선택 [single]:
```

- **multi**: 역할이 분리된 4명의 teammate 에이전트 생성
- **single**: 한 에이전트가 역할을 번갈아 수행 (역할 분리는 유지)

### 1.4 루프 모드

```text
[STDD] 루프 모드:

| 모드   | 설명                                   |
|--------|----------------------------------------|
| auto   | 통과까지 자동 반복 (최대 5회)            |
| semi   | 매 루프마다 진행 확인 (권장)             |
| manual | 각 단계마다 승인 필요                   |

선택 [semi]:
```

## Step 2: 작업 단위 식별

plan에서 구현 단위를 결정합니다:

- **Normal 모드**: `plan.md`의 Phase
- **Pro 모드**: `tasks/planned/`의 Work Package

진행 상태 표시:

```text
STDD 구현:
├─ 단위 1: {이름} - ○ 대기
├─ 단위 2: {이름} - ○ 대기
└─ 단위 3: {이름} - ○ 대기
```

## Step 3: 단위별 STDD 사이클

각 단위에 대해 5단계 STDD 사이클을 실행합니다:

### Stage 1: 명세 정의 (Define Spec)

`plan.md` 또는 워크 패키지에서 산출물과 수용 기준을 추출합니다.
Zero-Ambiguity 기술 명세로 변환합니다:

- **목적 (Purpose)**: 이 단위가 전달하는 것
- **입력 (Input)**: 작업할 기존 코드, 파일, API
- **출력 스키마 (Output Schema)**: 예상 파일, 타입, API 형태
- **제약사항 (Constraints)**: 기술, 패턴, 컨벤션
- **금지사항 (Prohibitions)**: 하지 말아야 할 것
- **에러 케이스 (Error Cases)**: 엣지 케이스와 예상 동작

**규칙:**

- 모든 기준은 O/X(통과/실패)로 검증 가능해야 함
- 주관적 표현 금지 ("좋은", "깔끔한", "적절한")
- 불확실한 부분은 명시적으로 표시

`$FEATURE_DIR/stdd/unit-{N}-spec.md`에 저장합니다.

사용자에게 보여주고 승인을 받은 후 진행합니다.

### Stage 2: 테스트 정의 (Test-First)

확정된 명세를 기반으로 구현 **전에** 테스트 기준을 작성합니다:

**기계적 테스트:**

- 문법 검사 (파싱 가능한가?)
- 린트/포맷 검사 (코딩 규칙 준수?)
- 타입 검사 (타입 오류 없는가?)

**실행 테스트:**

- 단위 테스트 (개별 함수/모듈 동작)
- 통합 테스트 (컴포넌트 간 상호작용)
- E2E 테스트 (전체 흐름, 필요 시)

**스키마 테스트:**

- API 응답 형식이 명세와 일치하는가?
- 데이터 타입이 정확한가?
- 에러 응답 형식이 명세와 일치하는가?

각 테스트에 ID 부여: T1, T2, T3...

`$FEATURE_DIR/stdd/unit-{N}-tests.md`에 저장합니다.

사용자에게 확인을 받습니다.

### Stage 3: 실행 (Execute)

명세와 테스트를 목표로 단위의 산출물을 구현합니다.

- **multi** 모드: coder 에이전트에게 [SPEC] + [TEST-CRITERIA] 전달
- **single** 모드: Coder 관점으로 전환 — 명세 기반으로만 구현, 테스트 구현 세부사항 무시

커밋 메시지:

- Normal 모드: `[Phase {N}] {이름}`
- Pro 모드: `[WP##] {이름}`

### Stage 4: 테스트 & 리팩터 루프

Stage 2의 모든 테스트를 실행합니다. 결과 보고:

```text
[STDD 테스트 보고서 - 단위 {N}, Loop #{L}]

PASSED:
- T1: {설명} ✅
- T3: {설명} ✅

FAILED:
- T2: {설명} ❌
  Expected: {기대값}
  Actual: {실제값}
  Action: {구체적 수정 지시}

SUMMARY: {통과}/{전체} passed.
```

**루프 동작:**

- **전체 통과** → Stage 5로 진행
- **실패 존재** → 에러 로그를 coder에게 전달 → 수정 → 재테스트
- **동일 에러 2회 연속** → 사용자에게 에스컬레이션
- **최대 루프 도달** → 중단 및 현재 상태 보고

**회귀 감지:**

동일 테스트가 반복 실패하는데 코드가 명세를 따르고 있으면 회귀를 제안합니다:

```text
[STDD 회귀 제안 - Loop #{L}]

테스트/명세 불일치가 의심됩니다.

근거: {구체적 근거}

선택:
(A) Stage 2로 회귀 - 테스트를 명세에 맞게 수정
(B) Stage 1로 회귀 - 명세 자체를 재검토
(C) 무시하고 루프 계속
```

각 루프 결과를 `$FEATURE_DIR/stdd/unit-{N}-loop-{L}.md`에 저장합니다.

### Stage 5: 단위 승인

결과를 제시합니다:

```text
[STDD 단위 {N} 완료: {이름}]

테스트: {통과}/{전체} passed
루프: {L}회
변경 파일: {목록}

이 단위를 승인하시겠습니까? (Y/N)
```

- **Y**: 완료 표시, 다음 단위로 진행
- **N**: 사용자 피드백 → 적절한 stage로 복귀

walkthrough 생성: `$FEATURE_DIR/walkthrough-phase-{N}.md` (Normal)
또는 `$FEATURE_DIR/walkthrough.md` (Pro).

## Step 4: 전체 완료

```text
✓ STDD로 모든 단위 구현 완료.

요약:
  단위 1: {이름} — {tests}개 테스트, {loops}회 루프 ✓
  단위 2: {이름} — {tests}개 테스트, {loops}회 루프 ✓
  ...

총계: {total_tests}개 테스트 통과, {total_loops}회 루프

다음: /spec-mix.merge
```

## 멀티에이전트 구성 (`multi` 선택 시)

`stdd-development` 팀을 4명의 에이전트로 생성합니다:

**stdd-spec-analyst:**

- 요구사항을 Zero-Ambiguity 명세로 변환
- 명확화 질문 (최대 3개)
- spec.md + 테스트 시나리오 초안 생성
- 참조: `docs/stdd/learnings/spec-analyst.md`

**stdd-coder:**

- 명세에 따라 코드를 정확히 구현
- tester의 테스트 코드를 먼저 확인
- 명세에 없는 기능을 임의로 추가하지 않음
- 에러 로그 수신 시 최소 범위로 수정
- 참조: `docs/stdd/learnings/coder.md`

**stdd-tester:**

- 구현 전 테스트 코드 먼저 작성 (Test-First)
- 명세 기준으로만 판정, coder의 의도 무시
- 표준 에러 로그 형식으로 보고
- 참조: `docs/stdd/learnings/tester.md`

**stdd-loop-driver:**

- tester의 에러 로그를 coder에게 전달
- 루프 횟수 및 안전 제한 추적
- 테스트/명세 불일치 감지 → 회귀 제안
- 반복 실패 시 에스컬레이션
- 참조: `docs/stdd/learnings/loop-driver.md`
