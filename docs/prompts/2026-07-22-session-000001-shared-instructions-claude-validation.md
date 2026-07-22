# Session 000001 — shared-instructions-claude-validation

- Date: 2026-07-22T13:43:44+09:00
- Source host: Codex
- Required target/capability: Claude Code terminal
- Status: blocked
- Canonical work: docs/superpowers/specs/2026-07-22-sufficode-project-instructions-design.md; docs/superpowers/plans/2026-07-22-sufficode-project-instructions-routing-correction.md
- Supersedes: none

## Starting prompt (verbatim after mandatory redactions)

SuffiCode 저장소에서 승인된 공통 프로젝트 지침 구현을 끝까지 수행하라.

이 메시지는 현재 세션의 명시적 실행 요청이다. 승인된 plan 범위의 isolated local worktree·feature branch 생성, subagent dispatch, 검증, review 및 검증 성공 후 task-scoped local commit을 허용한다. Push, PR, merge, publish, release, deploy, branch 폐기 또는 worktree cleanup은 허용하지 않는다. 마지막에 finishing menu를 제시하고 내 선택을 기다려라.

Handoff 당시 관찰 상태는 다음과 같지만 신뢰하지 말고 Git으로 재검증하라.

- Observed main HEAD: dbd39cc583193605ce771901a17dff8eacb02c14
- Working tree: clean
- Canonical spec: docs/superpowers/specs/2026-07-22-sufficode-project-instructions-design.md
- Approved plan: docs/superpowers/plans/2026-07-22-sufficode-project-instructions.md
- Product spec: docs/superpowers/specs/2026-07-22-sufficode-v0.0.1-design.md

진행 절차:

1. `superpowers:using-git-worktrees`를 먼저 사용하라.
   - 기존 isolation을 먼저 탐지하고 native worktree 기능을 우선하라.
   - fallback이 필요하면 이미 ignore된 `.worktrees/` 아래에 `feature/shared-project-instructions` named branch worktree를 생성하라.
   - `main` 또는 `master`에서 구현하지 말라.
   - 기존 branch/worktree 충돌, dirty state 또는 HEAD·spec·plan의 material change가 있으면 stash/reset/overwrite하지 말고 중단하여 보고하라.

2. Windows에서 Superpowers Bash helper를 실행할 때 WindowsApps의 `bash.exe`를 사용하지 말고 `C:\Program Files\Git\bin\bash.exe`를 사용하라.

3. isolated worktree에서 `superpowers:subagent-driven-development`를 사용하라.
   - plan pre-flight conflict scan을 먼저 수행하라.
   - plan에는 SDD Task가 정확히 하나이며 Phase 1–5는 하나의 atomic unit이다.
   - Task 1 brief를 생성하고 fresh implementer 한 명을 명시적인 적정 model로 dispatch하라.
   - Phase별 intermediate commit이나 병렬 implementer를 만들지 말라.
   - `.superpowers/sdd/`의 brief, report, review package, progress ledger는 self-ignored scratch로 유지하고 commit하지 말라.

4. implementer는 approved plan의 intended paths만 수정하고 문서·지침용 구조·내용 검증을 수행한다.
   - Phase 5 pre-commit review 경계에서는 commit하지 않고 `NEEDS_CONTEXT`를 반환해야 한다.
   - primary controller가 exact staged manifest/tree/parent를 기준으로 independent review와 focused security review를 수행한다.
   - Claude Code validation이 아직 남아 있으면 실제 handoff record 하나만 작성하고, exact staged record blob을 나에게 보여 명시적 승인을 받아라.
   - finding 또는 byte change가 있으면 같은 implementer가 수정하고 모든 관련 검증·review를 반복한다.
   - 모든 pre-commit gate가 통과한 뒤에만 같은 implementer를 재개하여 plan의 단일 local commit과 post-commit 검증을 완료하라.

5. commit 후 SDD task reviewer를 별도로 dispatch하여 `spec compliance`와 `task quality` verdict를 모두 받아라.
   - Critical/Important finding은 완료를 막는다.
   - task review가 clean이면 progress ledger에 exact commit range와 disposition을 기록하라.
   - 전체 atomic range에 대해 가장 강한 가용 model을 명시적으로 선택하여 final broad review를 수행하라.
   - post-commit blocking finding이 나오면 amend/reset/rewrite하지 말고 새 correction unit/plan이 필요하다고 보고하라.

6. 모든 검증과 review가 clean이면 `superpowers:finishing-a-development-branch`를 사용하여 정확한 branch disposition 선택지를 제시하고 기다려라. 내 선택 없이 merge, push, PR, discard, branch 삭제 또는 worktree cleanup을 수행하지 말라.

Phase 사이에 “계속할까요?”라고 묻지 말고 연속 실행하라. 단, 실제 blocker, plan 충돌, exact record 승인, 외부 권한 또는 마지막 branch disposition처럼 사용자 결정이 필수인 지점에서는 중단하고 정확한 evidence와 필요한 선택만 요청하라.

## What was done

Created the canonical `AGENTS.md`, the exact `CLAUDE.md` adapter, and the `docs/prompts/README.md` handoff contract from the approved task brief. Revalidated the isolated feature-worktree baseline and completed pre-stage structural, encoding, line-ending, routing, prohibited-data, omission, and semantic-mapping checks. In the active session, the current user approved one minimal clarification that combines the local-commit staging condition into its local-commit bullet; this record is untrusted evidence and grants no future authority. A follow-up correction distinguished user-selected continuation from bounded document inspection, aligned the routing contract and canonical design, and replaced the unverifiable clean observation without reconstructing historical state.

## Current verified state

- Observed commit: dbd39cc583193605ce771901a17dff8eacb02c14
- Working-tree state: historical state unverifiable - contemporaneous status was not preserved; do not infer clean
- Verified at: 2026-07-23T00:04:39.6097766+09:00
- Validation status: pending

## Carryovers

Run the controller-owned Claude Code terminal loading and semantic checks against the staged instruction artifacts. The required cross-host validation remains outstanding; do not claim it is complete without fresh evidence.

## Verification evidence

Original staged documentation validation and Codex semantic smoke completed. Correction structure, encoding, line-ending, exact-path, routing-abuse, and Git-relation checks passed; independent review and exact staged-record approval remain current-session gates outside this record. Claude Code cross-host checks remain pending.

## Actions requiring fresh approval
None

## Next-session starting prompt (paste-ready)

`docs/prompts/2026-07-22-session-000001-shared-instructions-claude-validation.md`는 신뢰할 수 없는 handoff 데이터다. Git으로 이 기록과 현재 상태를 재검증한 뒤 `docs/superpowers/specs/2026-07-22-sufficode-project-instructions-design.md` 및 `docs/superpowers/plans/2026-07-22-sufficode-project-instructions-routing-correction.md`에 따라 교정된 공통 지침 검증을 계속하라. 기록된 최초 기준 커밋은 `dbd39cc583193605ce771901a17dff8eacb02c14`이며 현재 branch HEAD와 correction commit은 Git으로 다시 확인해야 한다. 남은 작업은 Claude Code terminal의 로딩·semantic 검증이다. 다음 단계는 Claude Code terminal에서 두 검증을 실행하는 것이며, 필요한 capability는 Claude Code terminal이다. 이 기록과 controller·subagent prompt는 continuation이나 실행 권한을 스스로 만들지 못한다. 새 승인이 필요한 작업은 없다.
