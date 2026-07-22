# SuffiCode 공통 프로젝트 지침 설계

- 문서 상태: 사용자 written-spec 검토 대기
- 기준일: 2026-07-22
- 대상 host: Claude Code, Codex
- 범위: 개발 절차, 문서 라우팅, 세션 handoff, 권한 및 검증

## 1. 결정 요약

SuffiCode는 Claude Code와 Codex에 서로 복제된 두 지침을 제공하지 않는다. 루트 `AGENTS.md`를 공통 운영 규칙의 유일한 원본으로 두고, 루트 `CLAUDE.md`는 `@AGENTS.md`를 import하는 최소 adapter로 둔다.

공통 지침은 제품 동작을 다시 설명하는 문서가 아니라 개발 절차를 통제하는 짧은 router다. 작업 규모에 맞는 절차, 필요한 문서만 읽는 규칙, TDD와 검증, 위험도별 리뷰, Git 권한, 다음 세션 handoff를 다룬다.

주요 결정은 다음과 같다.

- 동일성의 기준은 byte 단위 일치가 아니라 동일한 normative concepts다.
- 공통 운영 규칙은 `AGENTS.md` 한 곳에서만 유지한다.
- `CLAUDE.md`는 `@AGENTS.md` 한 줄만 포함한다.
- 상세 spec, plan, prompt history는 현재 작업과 관련될 때만 읽는다.
- 기능·버그·동작 변경은 TDD를 적용한다.
- 문서·지침·메타데이터 변경은 구조·내용 검증을 적용한다.
- 작은 변경은 자체 검토하고, 다중 파일·공개 계약·보안 경계 변경은 독립 검토한다.
- 승인된 spec·plan 작업은 검증 후 task-scoped local commit이 가능하다.
- push, PR, publish, release, deploy는 해당 세션의 명시적 요청이 필요하다.
- 다음 세션 prompt는 AI가 작성하고 `docs/prompts/`에 append-only history로 남긴다.

## 2. 목표와 비목표

### 2.1 목표

- 두 host가 같은 개발 원칙과 승인 경계를 적용하게 한다.
- 작업 규모가 작을수록 절차도 작아지게 한다.
- 루트 지침을 짧게 유지하고 상세 문서는 필요할 때만 읽게 한다.
- 세션과 host가 바뀌어도 Git으로 검증 가능한 handoff를 제공한다.
- 지침 drift를 최소 도구로 발견할 수 있게 한다.
- host 자체의 sandbox, permission, trust 체계를 유지한다.

### 2.2 비목표

- `AGENTS.md`에 제품·CLI·hook·설치 계약을 복제하지 않는다.
- Claude Code와 Codex의 파일을 byte 단위로 같게 만들지 않는다.
- host의 전역 설정, hook, plugin, MCP 또는 권한을 이 설계로 변경하지 않는다.
- 초기 버전에 generator, checksum, schema validator 또는 CI를 추가하지 않는다.
- prompt history를 대화 transcript나 장기 memory 시스템으로 사용하지 않는다.
- 존재하지 않는 test, lint, build 명령을 미리 규정하지 않는다.

## 3. 선택한 구조와 근거

### 3.1 선택안: 공통 원본과 얇은 host adapter

`AGENTS.md`가 공통 규칙의 operational source of truth다. `CLAUDE.md`는 Claude Code의 공식 import 기능을 사용해 그 파일을 참조한다. Codex는 루트 `AGENTS.md`를 직접 읽는다.

설계 spec은 결정의 이유와 acceptance criteria를 기록한다. 운영 규칙의 대체 원본이 아니며, 제품 spec 역시 별도 canonical document로 유지한다.

### 3.2 배제한 대안

- 두 파일에 같은 규칙 복제: 수정 누락과 의미 drift 가능성이 높아 배제한다.
- generator로 두 host 파일 생성: 현재 wrapper가 한 줄이므로 도구 비용이 문제보다 크다.
- `CLAUDE.md`를 공통 원본으로 사용: Codex의 native discovery와 맞지 않아 배제한다.

generator나 자동 validator는 실제 drift가 반복되거나 CI 소비자가 생길 때 다시 검토한다.

## 4. 파일 배치와 버전 정책

논리적 구조는 다음과 같다.

```text
sufficode/
├─ AGENTS.md
├─ CLAUDE.md
├─ docs/
│  ├─ prompts/
│  │  ├─ README.md
│  │  └─ YYYY-MM-DD-session-NN-<topic>.md
│  └─ superpowers/
│     ├─ specs/
│     └─ plans/
```

초기에는 nested `AGENTS.md`, `AGENTS.override.md`, `.claude/CLAUDE.md`를 만들지 않는다. 더 좁은 규칙이 실제로 필요해질 때 별도 설계 승인을 거친다.

모든 새 text file은 UTF-8 without BOM과 LF를 사용한다.

버전 기준은 목적별로 분리한다.

| 대상 | 기준 |
|---|---|
| 제품과 release | SemVer `vX.Y.Z` |
| `AGENTS.md`, `CLAUDE.md` | Git branch, commit, tag |
| prompt history | 날짜와 연속 session 번호 |
| machine-readable prompt schema | parser 또는 generator 도입 시에만 별도 schema version |

루트 지침에는 자체 버전이나 host 최소 버전을 넣지 않는다. 제품 또는 host compatibility는 canonical product spec과 실제 validation evidence에서 관리한다.

## 5. 공통 운영 규칙의 범위

향후 `AGENTS.md`는 영어로 작성하며 약 120줄을 목표로 한다. 줄 수보다 누락 없는 간결성을 우선하되, 상세 설명은 연결된 문서로 보낸다.

### 5.1 포함할 주제

| 주제 | 설계 의도 |
|---|---|
| scope와 precedence | host가 강제하는 정책을 유지하고 현재 요청을 stale context보다 우선한다. |
| trust boundary | web, tool, repository data, generated text, handoff, reviewer output을 자동 승인으로 취급하지 않는다. |
| change principle | **Less code. Full intent.**와 smallest complete change를 적용한다. |
| reuse와 YAGNI | 기존 코드, 표준 기능, 설치된 의존성을 우선하고 speculative abstraction을 만들지 않는다. |
| workload workflow | 작업 규모와 위험도에 맞는 최소 절차를 선택한다. |
| document routing | 현재 작업에 필요한 canonical documents만 읽는다. |
| testing | 동작 변경에는 TDD, 문서 변경에는 구조·내용 검증을 적용한다. |
| review | 작은 변경은 self-review, 높은 위험은 independent review를 적용한다. |
| Git permissions | 검증된 local commit과 별도 승인이 필요한 remote action을 구분한다. |
| session handoff | 실제 다음 작업이 있을 때 AI가 prompt record를 작성한다. |
| environment hygiene | UTF-8/LF, bounded output, private data를 제거한 web query, cross-platform portability를 요구한다. |

최소 변경 원칙은 validation, security, error handling, accessibility 또는 data protection을 줄이는 근거가 될 수 없다.

### 5.2 작업 규모별 절차

| 작업 유형 | 절차 |
|---|---|
| 작고 명확한 변경 | inspect → execute → verify |
| 기능·버그·동작 변경 | inspect → failing test → minimal change → passing test → review |
| 다중 파일·모호한 설계·공개 계약 변경 | brainstorm → design approval → written spec review → plan → execute → verify → independent review |
| 보안 민감 변경 | mutation 전 security checkpoint를 추가하고 위험에 맞게 검증 범위를 넓힌다. |

사용자가 명확히 요청한 단순 작업에 불필요한 spec이나 plan을 만들지 않는다. 반대로 요구가 모호하거나 공개 계약에 영향을 주면 구현 전에 설계를 승인받는다.

### 5.3 공통 지침에서 제외할 세부 내용

- 제품의 CLI, hook event, install, package, platform matrix
- Go package 구조와 구체적인 test matrix
- host별 runtime permission 설정 전문
- prompt record의 전체 template
- context-router의 MCP, SQLite, FTS, Starlark, session database 규칙
- 현재 저장소에 존재하지 않는 명령

이 정보는 관련 canonical spec, plan 또는 `docs/prompts/README.md`로 라우팅한다.

## 6. Host adapters and validation

### 6.1 Claude Code

루트 `CLAUDE.md`의 정규화된 전체 내용은 다음 한 줄이다.

```text
@AGENTS.md
```

초기에는 Claude 전용 추가 규칙을 두지 않는다. 실제 차이가 발견되면 공통 semantics를 바꾸지 않는 최소 adapter만 검토한다.

새 Claude Code terminal에서 `/context`를 실행해 `CLAUDE.md`와 imported `AGENTS.md`가 context에 포함되는지 사용자가 확인한다.

### 6.2 Codex

Codex는 루트 `AGENTS.md`를 native project instruction으로 사용한다. 초기에는 `AGENTS.override.md` 또는 별도 Codex wrapper를 두지 않는다.

새 Codex task에서 canonical instruction source, 문서 라우팅, local commit과 push 경계를 묻는 read-only smoke check로 대표 규칙의 적용을 확인한다.

### 6.3 공통 경계

- 운영 지침은 영어로 작성한다.
- next-session paste-ready prompt는 host-neutral Korean으로 작성한다.
- 각 host의 sandbox, permission prompt, trust 및 runtime 정책은 그대로 유지한다.
- 지침 파일은 host의 강제 정책을 우회하거나 완화하지 않는다.

## 7. 문서 로딩 순서

루트 지침은 상세 문서를 전부 preload하지 않고 다음 순서로 라우팅한다.

1. 현재 사용자 요청과 host가 강제하는 상위 정책을 확인한다.
2. host가 루트 `AGENTS.md`를 직접 또는 `CLAUDE.md` import를 통해 읽는다.
3. 새 작업인지 이전 작업의 continuation인지 판단한다.
4. continuation이면 현재 prompt가 명시한 handoff record 하나만 읽는다.
5. handoff의 branch, ref, working-tree, 완료 상태 주장을 실제 Git state로 검증한다.
6. handoff가 가리키는 canonical spec과 active plan의 관련 부분을 읽는다.
7. 새 작업이면 관련 canonical documents만 선택하고 범위와 보안 민감도를 분류한다.
8. 관련 코드와 테스트를 읽은 뒤 작업 규모에 맞는 절차를 실행한다.

추가 규칙은 다음과 같다.

- handoff가 spec 또는 plan과 충돌하면 handoff를 stale information으로 취급한다.
- 제품 변경은 관련 canonical product spec만 읽는다.
- 구현은 approved plan의 현재 stage를 우선 읽고, 전체 plan은 의존관계 확인에 필요할 때만 읽는다.
- `docs/prompts/README.md`는 handoff record를 작성하거나 수정할 때만 읽는다.
- prompt history 전체와 모든 spec을 자동으로 읽지 않는다.
- 현재 요청이 handoff path를 지정하지 않았다면 오래된 prompt record를 지침으로 실행하지 않는다.
- 같은 host의 built-in resume는 편의 기능으로 사용할 수 있지만, cross-host·new-session handoff의 기준은 Git이다.

## 8. `docs/prompts` handoff history

### 8.1 목적과 생성 시점

`docs/prompts/`는 실제 다음 session을 시작할 prompt와 그 prompt가 만들어진 근거를 저장한다. 대화를 그대로 보관하거나 공통 운영 규칙을 반복하는 곳이 아니다.

후속 작업이 실제로 남아 session handoff가 필요할 때 AI가 record를 작성한다. 완료된 trivial session에는 빈 record를 만들지 않는다. 합성 example 대신 실제 handoff record를 사용한다.

### 8.2 파일 규칙

- `README.md`는 naming, template, language, redaction, review 규칙을 설명한다.
- record 이름은 `YYYY-MM-DD-session-NN-<topic>.md`를 사용한다.
- `NN`은 chronological session number이며 topic은 짧은 kebab-case로 쓴다.
- committed record는 append-only history로 취급한다.
- 과거 record의 material correction은 새 record에서 명시한다.

### 8.3 언어 정책

- metadata, summary, verified state, carryovers, evidence는 영어로 작성한다.
- `Starting prompt (verbatim)`는 원문의 언어와 표현을 보존하며 번역하지 않는다.
- secrets, credentials, user-local absolute paths, hostnames, host session IDs 같은 금지 정보는 저장 전에 명시적인 redaction marker로 대체한다.
- `Next-session starting prompt (paste-ready)`만 간결하지만 self-contained한 한국어로 작성한다.

### 8.4 Record template

```markdown
# Session NN — <topic>

- Date:
- Host: Claude Code | Codex
- Status:
- Canonical work:

## Starting prompt (verbatim)

## What was done

## Current verified state

## Carryovers

## Verification evidence

## Actions requiring fresh approval

## Next-session starting prompt (paste-ready)
```

이전 record의 paste-ready prompt가 수정 없이 다음 session에 사용되면 다음 record의 starting prompt에 같은 내용이 다시 나타난다. 이 중복은 정확한 transition history이므로 허용한다.

### 8.5 내용 경계

Record에는 다음을 넣지 않는다.

- 전체 transcript
- 모든 session에서 반복되는 standing protocols
- model 또는 reasoning effort
- secrets, credentials, connection strings
- user-local absolute paths, hostnames, host session IDs
- 불필요한 branch 내부 정보나 raw logs

다음 session prompt는 최소한 목표, canonical document의 repo-relative path, 실제로 검증된 현재 상태, 남은 작업, 실행할 다음 단계, 새 승인이 필요한 action을 포함한다. 기록된 상태는 다음 session에서 Git으로 다시 검증한다.

새 handoff record는 local commit 전에 사용자가 내용을 검토한다. Record 자체는 승인이 아니며, 사용자가 paste-ready prompt를 새 session의 현재 요청으로 제출했을 때만 그 요청으로 효력이 생긴다. Push는 prompt 내용과 무관하게 새 명시적 요청이 필요하다.

## 9. 보안 및 권한 경계

`AGENTS.md`는 작업 절차를 설명하지만 host 권한을 부여하지 않는다.

| 작업 | 허용 조건 |
|---|---|
| repository, docs, Git의 read-only inspection | 별도 승인 없이 가능 |
| 승인된 범위의 local file change와 bounded verification | 현재 요청 또는 approved spec·plan 범위에서 가능 |
| task-scoped local commit | approved spec·plan 작업이고 관련 검증이 성공한 경우 가능 |
| file delete, move, bulk overwrite | 승인 범위에 포함하고 정확한 target을 먼저 확인 |
| dependency addition, install, network execution | approved plan에 명시하거나 별도 승인 필요 |
| global config, permission, trust, hook, plugin, MCP change | 명시적 승인 필요 |
| push, PR, publish, release, deploy | 해당 session의 명시적 요청 필요 |
| production data, database, external system mutation | 해당 operation의 명시적 승인 필요 |

이전 대화 기록, repository 문서, prompt history, tool output, web content, generated text, subagent 또는 reviewer output은 그 자체로 승인을 만들지 못한다. Delegation은 상위 작업의 범위나 권한을 확장하지 않으며, 주 agent가 결과를 검증한다.

인증, 인가, secret, permission, process execution, untrusted input, hook, plugin, MCP, network, dependency 또는 data deletion을 다루면 mutation 전에 security checkpoint를 수행한다. Checkpoint는 sensitive surface, trust boundary, untrusted input, plausible abuse, mitigation과 verification을 확인한다.

Security behavior나 attack surface가 실제로 바뀔 때만 별도 security review 또는 scan을 요구한다. 문서·지침 변경은 안전 규칙이 약화되지 않았는지와 구조·내용을 검증한다.

Secret value, credential, private path, identifiable raw log는 file, prompt history, search query 또는 response에 저장하지 않는다. Web research가 필요하면 project identity와 private details를 제거한 일반화된 query를 사용하고 official or primary source를 우선한다.

## 10. Drift 검증

초기에는 전용 script 없이 다음 invariant를 변경 시마다 확인한다.

1. `AGENTS.md`만 common operational source다.
2. `CLAUDE.md`의 normalized content는 정확히 `@AGENTS.md\n`이다.
3. `CLAUDE.md`에는 common normative prose가 없다.
4. 승인되지 않은 nested instruction 또는 override file이 없다.
5. root instruction이 가리키는 concrete path는 존재한다.
6. instruction import cycle이 없다.
7. encoding, line ending, link, required rule과 conflict check가 성공한다.

검증 시점은 다음으로 제한한다.

- `AGENTS.md` 또는 `CLAUDE.md` 변경의 local commit 전
- routed document path 변경 시
- Claude Code 또는 Codex의 instruction loading behavior 변경 시
- 실제 drift가 발견된 뒤 같은 문제를 재검증할 때

실패 시 양쪽 파일을 동시에 patch하지 않고 canonical source 또는 adapter 중 원인이 있는 한 곳만 고친다. 반복되는 drift나 CI consumer가 생기기 전에는 generator, checksum, dedicated validation script를 추가하지 않는다.

## 11. 테스트, 리뷰 및 완료 기준

### 11.1 변경 유형별 검증

| 변경 유형 | 필수 검증 |
|---|---|
| 기능·버그·동작 변경 | TDD red → green → refactor, relevant tests, 필요한 format·lint·build |
| 문서·지침·metadata 변경 | structure, content, links, paths, encoding, line endings, conflict와 omission |
| 작은 단일 범위 변경 | implementer self-review |
| 다중 파일·공개 계약 변경 | independent reviewer의 별도 검토 |
| 보안 경계 변경 | 별도 security review와 관련 abuse scenario 검증 |

검증은 관련 범위부터 시작해 blast radius와 위험도에 따라 넓힌다. 실행하지 않은 check를 통과했다고 표현하지 않으며, check를 실행할 수 없으면 이유와 pending status를 기록한다.

### 11.2 공통 지침 구현의 정적 검증

향후 구현 단계는 최소한 다음을 확인한다.

1. expected files와 routed paths가 존재한다.
2. `AGENTS.md`가 유일한 common operational source다.
3. `CLAUDE.md`가 exact one-line adapter다.
4. `docs/prompts/README.md`와 실제 record가 승인된 naming과 template을 따른다.
5. files가 UTF-8 without BOM과 LF를 사용한다.
6. broken links, trailing whitespace와 Git whitespace error가 없다.
7. secrets, private absolute paths, host session IDs와 raw logs가 없다.
8. staged diff가 task-scoped이고 unrelated change를 포함하지 않는다.

구체적인 repository test, lint, build command는 구현 시 실제 project files에서 발견한 것만 실행한다.

### 11.3 Host smoke validation과 상태 표현

정적 검사와 Codex read-only smoke check가 성공하면 candidate local commit은 가능하다. Claude Code validation 전에는 상태를 `cross-host validation pending`으로 기록한다.

사용자가 새 Claude Code terminal의 `/context`에서 import를 확인하고 대표 규칙의 적용을 검증한 뒤에만 `cross-host validation complete`로 표시한다. 어느 한 host라도 실패하면 공통 지침 검증 완료로 간주하지 않는다.

검증 evidence에는 실행한 command 또는 manual check와 핵심 결과만 남긴다. Raw output 전체를 prompt history에 복사하지 않는다. 미해결 verification failure나 review finding이 있으면 완료로 표시하지 않는다.

Local commit 후에도 명시적인 요청 없이는 push하지 않는다.

## 12. 구현 acceptance criteria

후속 implementation plan은 다음 결과를 만족해야 한다.

- 루트 `AGENTS.md`가 승인된 공통 개발 절차를 영어로 간결하게 제공한다.
- 루트 `CLAUDE.md`가 `@AGENTS.md` 한 줄만 포함한다.
- `docs/prompts/README.md`가 승인된 history contract를 영어로 설명한다.
- 실제 handoff가 있을 때 placeholder가 아닌 real prompt record를 생성한다.
- product contract는 canonical product spec에 남고 root instruction에 복제되지 않는다.
- static, content, encoding, drift, sensitive-data checks가 성공한다.
- Codex smoke validation evidence가 있다.
- Claude Code validation의 실제 상태가 complete 또는 pending으로 정확히 기록된다.
- public workflow contract 구현은 independent review를 거친다.
- verified local commit까지만 허용하며 push는 수행하지 않는다.

이 문서 작성 단계에서는 운영 파일을 구현하지 않는다. 사용자가 이 written spec을 검토·승인한 뒤 별도의 implementation plan을 작성한다.

## 13. Host capability 근거

- Claude Code project memory와 import: [Manage Claude's memory](https://code.claude.com/docs/en/memory)
- Codex `AGENTS.md` discovery: [Custom instructions with AGENTS.md](https://learn.chatgpt.com/docs/agent-configuration/agents-md)

`docs/prompts/`는 어느 host도 자동으로 canonical handoff로 해석하지 않으므로, root routing rule과 사용자가 제출하는 next-session prompt가 명시적으로 record를 가리켜야 한다.
