---
description: Execute implementation based on current mode (Normal or Pro)
---

## Mode Detection

Check mode from `.spec-mix/config.json`:

```bash
cat .spec-mix/config.json 2>/dev/null | grep '"mode"' || echo '"mode": "normal"'
```

## Mode-Specific Workflow

### If `mode: "normal"` (Default)

**Phase-based implementation with built-in review:**

1. Execute one phase at a time
2. Generate walkthrough after each phase
3. Present review → Accept/Reject
4. Proceed after acceptance
5. Final: Run `/spec-mix.merge`

**See**: `/spec-mix.implement-normal` for detailed workflow

---

### If `mode: "pro"`

**Work Package lane workflow:**

```
planned → doing → for_review → done
```

1. Select task from `planned`, move to `doing`
2. Implement with commits tagged `[WP##]`
3. Move completed task to `for_review`
4. Generate walkthrough
5. Next: `/spec-mix.review` → `/spec-mix.accept` → `/spec-mix.merge`

**See**: `/spec-mix.implement-pro` for detailed workflow

---

### STDD 오버라이드 (모든 모드)

사용자가 인자에 `--stdd` 또는 `stdd`를 지정하면, 현재 모드와 관계없이 STDD 방법론을 사용합니다.

**STDD (Spec & Test Driven Development):**

1. 페이즈/태스크를 Zero-Ambiguity 기술 명세로 변환
2. 코드 작성 전 테스트 먼저 작성 (Test-First)
3. 모든 테스트를 목표로 구현
4. Error-as-Prompt 루프: 실패 → 수정 → 재테스트
5. 모든 테스트 통과 후 승인

**See**: `/spec-mix.implement-stdd` for detailed workflow

---

## Quick Reference

| Mode | Task Unit | Review | Commands After |
|------|-----------|--------|----------------|
| Normal | Phase | Built-in Accept/Reject | `/spec-mix.merge` |
| Pro | Work Package | `/spec-mix.review` | `/spec-mix.accept`, `/spec-mix.merge` |
| STDD | Phase/WP | 5-stage cycle per unit | `/spec-mix.merge` |
