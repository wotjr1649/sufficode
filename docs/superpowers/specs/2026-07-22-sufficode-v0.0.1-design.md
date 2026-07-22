# SuffiCode v0.0.1 통합 스펙·설계서

- 문서 상태: 사용자 written-spec 검토 대기
- 제품 상태: Developer Preview
- 기준일: 2026-07-22
- 저장소: `github.com/wotjr1649/sufficode`
- 슬로건: **Less code. Full intent.**

## 1. 제품 정의

SuffiCode는 Claude Code와 Codex가 사용자의 요구 범위를 줄이지 않으면서 가장 작은 완전한 변경을 만들도록 돕는 Go 기반 네이티브 정책 런타임이다.

SuffiCode는 다음 세 부분으로 구성한다.

1. 별도로 설치되는 단일 Go 실행 파일 `sufficode`
2. Claude Code용 marketplace와 얇은 plugin adapter
3. Codex용 marketplace와 얇은 plugin adapter

Plugin은 실행 파일을 포함하지 않는다. Hook는 host 프로세스의 `PATH`에서 이미 설치된 `sufficode`를 실행한다.

## 2. v0.0.1 목표

- Claude Code와 Codex에 동일한 SuffiCode 정책 의미를 제공한다.
- 새 세션은 기본 ON으로 시작한다.
- OFF는 현재 세션에만 적용한다.
- Windows, Linux, macOS의 확정된 64비트 네 대상에서 source install과 실행을 검증한다.
- 사용자 전역 marketplace/plugin 설치를 기존 설정을 보존하며 자동화한다.
- Hook 처리 중 네트워크, 텔레메트리, 자체 LLM 호출을 사용하지 않는다.
- 정책 주입 오버헤드를 작은 고정 kernel로 제한한다.

## 3. 명시적 비목표

v0.0.1에는 다음을 포함하지 않는다.

- MCP 서버
- Skill 또는 custom agent
- 다른 AI host
- `lite`, `full`, `ultra` 모드
- 자동 모델·reasoning effort 변경
- 사전 빌드 바이너리와 아카이브
- GoReleaser, Windows 코드 서명, macOS Developer ID
- daemon, GUI, status line, web dashboard
- 원격 텔레메트리와 자체 API key
- 자동 자연어 작업 분류
- diff·로그 축약 도구
- host 설정 JSON/TOML 직접 편집
- binary self-update와 self-delete

## 4. 지원 범위

### 4.1 Host

- Claude Code
- Codex CLI/App plugin runtime

실제 최소 지원 버전은 계약 테스트로 확정한다. 설계 조사에서 검증한 기준점은 다음과 같다.

- Claude Code `2.1.216`
- Codex CLI `0.144.6`

이 값은 구현 전 임시 하한이며 공식 최소 버전 선언은 CI와 실기 검증 후 확정한다.

### 4.2 운영체제와 아키텍처

| 사용자 표기 | Go target | GitHub Actions runner |
|---|---|---|
| Windows x86-64 | `windows/amd64` | `windows-2025` |
| Linux x86-64 | `linux/amd64` | `ubuntu-24.04` |
| Linux ARM64 | `linux/arm64` | `ubuntu-24.04-arm` |
| macOS Apple Silicon | `darwin/arm64` | `macos-15` |

공식 지원에서 제외한다.

- 모든 32비트 target
- `windows/arm64`
- `darwin/amd64`
- 그 밖의 GOOS/GOARCH

공식 표현은 “네 target의 source-install 및 테스트 지원”이다. 사전 빌드 바이너리 제공으로 표현하지 않는다.

### 4.3 Toolchain

- 최소 Go 버전: `1.26.5`
- 외부 runtime module dependency: 없음
- 기본 빌드: `CGO_ENABLED=0`
- 더 최신 Go compiler로 빌드되는 것은 허용한다.
- bit-for-bit compiler 재현성은 v0.0.1 목표가 아니다.

외부 CI 도구는 runtime dependency가 아니다. 표준 라이브러리만 사용하면 `go.sum`은 생성될 필요가 없다.

## 5. 저장소 구조

```text
sufficode/
├─ go.mod
├─ README.md
├─ LICENSE
├─ THIRD_PARTY_NOTICES.md
├─ UPSTREAM.md
│
├─ cmd/
│  └─ sufficode/
│     └─ main.go
│
├─ internal/
│  ├─ cli/
│  ├─ hook/
│  ├─ policy/
│  ├─ setup/
│  ├─ state/
│  └─ version/
│
├─ .agents/
│  └─ plugins/
│     └─ marketplace.json
│
├─ .claude-plugin/
│  └─ marketplace.json
│
├─ plugins/
│  └─ sufficode/
│     ├─ .codex-plugin/
│     │  └─ plugin.json
│     ├─ .claude-plugin/
│     │  └─ plugin.json
│     └─ hooks/
│        ├─ hooks.json
│        └─ claude.json
│
├─ testdata/
│  ├─ claude/
│  ├─ codex/
│  └─ setup/
│
├─ docs/
│  └─ superpowers/
│     ├─ specs/
│     │  └─ 2026-07-22-sufficode-v0.0.1-design.md
│     └─ plans/
│        ├─ 2026-07-22-sufficode-v0.0.1-core-runtime.md
│        └─ 2026-07-22-sufficode-v0.0.1-marketplace-setup-release.md
│
└─ .github/
   └─ workflows/
      ├─ verify.yml
      └─ release.yml
```

새 패키지는 실제 책임이 분리될 때만 추가한다. 공개 `pkg/`, clean architecture 계층, 단일 구현 interface는 만들지 않는다.

## 6. Marketplace와 plugin 계약

### 6.1 공통 식별자

```text
Marketplace name: sufficode-plugins
Plugin name:      sufficode
Plugin source:    ./plugins/sufficode
```

Marketplace entry에는 plugin version을 중복하지 않는다. Version은 두 plugin manifest에 기록하고 CI에서 Git tag와 일치하는지 검사한다.

### 6.2 Claude Code

- Marketplace: `.claude-plugin/marketplace.json`
- Plugin manifest: `plugins/sufficode/.claude-plugin/plugin.json`
- Hook manifest: `plugins/sufficode/hooks/claude.json`
- Plugin manifest가 `"hooks": "./hooks/claude.json"`을 명시한다.
- Hook command는 `command`와 `args`를 분리한 exec-form을 사용한다.

Marketplace source는 문자열 상대 경로 `./plugins/sufficode`를 사용한다.

### 6.3 Codex

- Marketplace: `.agents/plugins/marketplace.json`
- Plugin manifest: `plugins/sufficode/.codex-plugin/plugin.json`
- Hook manifest: `plugins/sufficode/hooks/hooks.json`
- `.codex-plugin/plugin.json`에는 v0.0.1에서 `hooks` 필드를 넣지 않는다.
- Codex의 기본 발견 경로 `hooks/hooks.json`을 사용한다.
- Hook command는 사용자 입력이나 host 필드를 보간하지 않는 정적 `command`/`commandWindows`만 사용한다.

Marketplace source는 local source object와 상대 경로 `./plugins/sufficode`를 사용한다.

### 6.4 Shared plugin root 원칙

- Plugin 설치·cache 경계는 `plugins/sufficode`이다.
- Plugin에서 저장소 루트의 Go source를 `../..`로 참조하지 않는다.
- `go run`으로 저장소 source를 실행하지 않는다.
- Plugin은 PATH의 `sufficode` 실행 파일만 호출한다.
- 두 marketplace는 host별 sparse checkout으로 상대 host의 root catalog를 제외한다.

Sparse path 집합:

```text
Codex:  .agents/plugins, plugins/sufficode
Claude: .claude-plugin, plugins/sufficode
```

Codex와 Claude validator가 shared root의 상대 host 파일을 거부하는 것이 실제 계약 테스트에서 확인될 때만 plugin root를 host별로 분리한다.

## 7. CLI 계약

사용자 명령:

```text
sufficode setup
sufficode doctor
sufficode uninstall
sufficode version
```

내부 hook 명령:

```text
sufficode hook claude --adapter-schema=1
sufficode hook codex --adapter-schema=1
```

`on`, `off`, `status`는 외부 CLI 명령이 아니라 활성 session ID를 가진 `UserPromptSubmit`의 정확한 전체 prompt 제어 명령이다.

### 7.1 종료 코드

```text
0: 요청한 작업이 완료됨
1: 작업 실패 또는 지원 host 없음
2: host별 부분 성공
```

Hook 프로세스의 malformed/oversized 입력은 advisory 정책이 host 작업을 막지 않도록 exit 0, context 없음으로 fail-open한다.

### 7.2 version

`runtime/debug.ReadBuildInfo`를 사용해 다음을 표시한다.

- binary version
- Go compiler version
- GOOS/GOARCH
- adapter schema 지원 범위

Version을 source에 하드코딩하지 않는다.

### 7.3 doctor

`doctor`는 read-only이다. Marketplace refresh, plugin update, 설정 수정, 네트워크 호출을 하지 않는다.

확인 항목:

- 실행 binary와 PATH 해석 결과
- 지원 GOOS/GOARCH
- release/devel build
- Claude/Codex 감지와 version 실행
- marketplace/plugin CLI capability
- installed/enabled/configured 상태
- Codex hook trust가 필요한 상태
- adapter schema 호환성
- ownership receipt와 실제 host 상태 차이
- 부분 설치 또는 수동 복구 필요 상태

실제 모델 호출 없이 `active`라고 단정하지 않는다.

## 8. 설치 계약

### 8.1 최초 설치

Go가 설치되어 있다는 전제에서 exact version을 사용한다.

```text
go install github.com/wotjr1649/sufficode/cmd/sufficode@v0.0.1
sufficode setup
```

첫 명령은 아직 존재하지 않는 `sufficode` 실행 파일을 설치한다. 두 번째 명령은 한 번의 실행으로 설치된 Claude Code와 Codex를 감지하고 각 host adapter를 등록한다.

문서의 안정 설치 경로에는 `@latest`를 사용하지 않는다.

### 8.2 setup 사전조건

- `runtime/debug.ReadBuildInfo`의 main version이 정식 `vX.Y.Z`이다.
- `(devel)`, unknown, pseudo-version은 신규 setup과 migration을 거부한다.
- `os.Executable()`과 `exec.LookPath("sufficode")`가 같은 실행 파일을 가리킨다.
- host를 단순 파일 존재가 아니라 `--version` 실행 성공과 plugin capability로 판별한다.
- host 프로세스가 동일 PATH를 받도록 setup 후 host를 재시작한다.

GUI/Desktop은 해당 프로세스가 PATH를 상속하고 실기 검증을 통과한 경우에만 지원으로 표기한다.

### 8.3 Host별 설치 transaction

각 host는 독립 transaction이다.

1. 현재 marketplace/plugin 상태 조회
2. 같은 이름의 다른 source/ref 충돌 검사
3. exact Git ref와 host별 sparse path로 marketplace 등록
4. Claude는 `plugin install`, Codex는 `plugin add`
5. 설치 상태를 JSON으로 재조회하여 검증
6. SuffiCode가 만든 항목만 ownership receipt에 기록

한 host 실패가 다른 host의 성공을 rollback하지 않는다. 일부 성공은 exit 2로 보고한다.

Codex hook trust는 사용자 보안 단계이다. `setup`은 trust를 우회하지 않고 `/hooks` 검토와 새 session 시작을 안내한다.

### 8.4 소유권과 기존 설정 보존

- Host 설정 JSON/TOML을 직접 수정하지 않는다.
- 기존 marketplace는 SuffiCode가 소유하지 않는다.
- 같은 marketplace 이름이 다른 source를 가리키면 무변경 실패한다.
- SuffiCode가 만든 marketplace라도 다른 plugin이 사용하면 uninstall에서 보존한다.
- Receipt가 없거나 손상되면 정확한 SuffiCode plugin만 제거하고 marketplace는 보존한다.
- Host CLI가 없거나 비호환이면 설정을 추측해 편집하지 않고 수동 복구 절차를 반환한다.

Receipt에는 secret, prompt, raw 설정을 저장하지 않는다. 다음 최소 metadata만 보관한다.

```text
host
marketplace name/source/ref
plugin name
adapter schema/revision
marketplaceCreatedBySufficode
createdAt
```

Receipt는 versioned append-only generation으로 쓰고 시작 시 실제 host 상태와 reconcile한다.

### 8.5 uninstall

- 지원 host CLI를 통해 SuffiCode plugin을 제거한다.
- SuffiCode가 소유한 marketplace만 제거 대상으로 고려한다.
- Binary 자체는 실행 중 삭제하지 않는다.
- `(devel)` build도 정확히 식별된 SuffiCode 항목의 doctor/uninstall을 허용한다.
- Binary 삭제는 host 종료 후 사용자가 수행한다.

## 9. 업데이트와 rollback

다음 값을 분리한다.

```text
binaryVersion
adapterSchema
adapterRevision
```

- v0.0.x 동안 `adapterSchema=1`을 동결한다.
- 호환성은 semver 동일성이 아니라 adapter schema로 판단한다.
- Policy와 동작은 Go binary에 embed한다.
- 같은 schema 내 binary update는 기존 adapter로 동작해야 한다.
- `setup`은 항상 멱등하게 재실행할 수 있지만 schema가 같으면 adapter 변경이 필수는 아니다.
- Schema가 바뀌는 release에서만 owned plugin/marketplace의 remove, exact ref 재등록, reinstall, 검증 migration을 수행한다.
- Migration 실패 시 이전 receipt를 이용해 best-effort 보상하고 `doctor`가 partial 상태를 표시한다.
- Hook CLI/wire contract는 적어도 v0.x 동안 이전 adapter와 호환한다.

Go module download와 Git marketplace clone은 별도 경로이므로 같은 tag 문자열만으로 암호학적 동일성을 주장하지 않는다.

Release 정책:

- GitHub immutable releases를 사용하거나 tag update/delete를 ruleset으로 금지한다.
- 공개된 tag를 이동하거나 재사용하지 않는다.
- 실패한 release는 새 patch version으로 수정한다.
- `go.mod`에 release용 `replace`와 `exclude`를 두지 않는다.

## 10. Hook runtime 계약

### 10.1 사용하는 event

| Event | 용도 |
|---|---|
| `SessionStart` | startup/resume/clear/compact 상태 처리와 필요한 정책 주입 |
| `UserPromptSubmit` | exact ON/OFF/STATUS 명령 및 pending 전이 처리 |
| `SubagentStart` | ON일 때 역할별 compact 정책 주입 |

`PostCompact`는 사용하지 않는다. Compact 후 정책 복구는 `SessionStart(source=compact)`에서 수행한다.

### 10.2 입력 신뢰 경계

Hook stdin의 모든 필드는 비신뢰 입력이다.

- 최대 입력: 1 MiB
- streaming JSON decode
- unknown field 허용
- 필수 event/type만 검증
- trailing JSON 거부
- 짧은 처리 deadline
- prompt는 exact control command 비교 외 사용하지 않음
- command line에 session ID, prompt, cwd, agent type을 보간하지 않음
- stdout에는 host protocol JSON만 출력
- diagnostics는 stderr의 고정 오류 코드만 사용
- prompt, raw session ID, 경로, 코드 내용을 log하지 않음

Malformed, oversized, incompatible schema, 내부 오류는 host 작업을 차단하지 않고 context 없이 fail-open한다.

### 10.3 네트워크 경계

Hook event 처리 중:

- network 0
- telemetry 0
- 자체 LLM 호출 0
- subprocess 추가 실행 0

설치와 migration은 Go module, GitHub, host marketplace CLI의 네트워크를 사용할 수 있다. SuffiCode 자체 telemetry와 외부 installer의 통신을 구분해 문서화한다.

## 11. Session ON/OFF 계약

### 11.1 사용자 명령

다음 세 문자열만 대소문자를 무시하고 양쪽 공백을 제거한 전체 prompt로 인식한다.

```text
sufficode on
sufficode off
sufficode status
```

부분 문자열, prefix, slash alias, 자연어 분류는 사용하지 않는다.

제어 prompt는 host별 documented block/suppress 응답으로 모델에 전달하지 않는다.

### 11.2 상태 전이

```text
ON --off--> OFF_PENDING --다음 일반 prompt/revoke--> OFF
OFF --on--> ON_PENDING  --다음 일반 prompt/policy--> ON
```

- `OFF_PENDING`의 revoke는 at-least-once이다. Crash로 반복되어도 의미가 변하지 않아야 한다.
- `ON_PENDING`의 policy 주입도 반복 안전해야 한다.
- `status`는 현재 논리 상태와 pending 여부만 표시한다.
- OFF는 기존 context를 삭제하지 못하므로 “이후 요청부터 정책을 적용하지 않음”을 의미한다.

### 11.3 lifecycle

| SessionStart source | 동작 |
|---|---|
| `startup` | marker가 없으면 ON으로 시작하고 정책 주입 |
| `resume` | 기존 상태 보존; ON이면 정책 주입, OFF이면 무주입 |
| `clear` | 기존 session marker 제거, ON으로 초기화하고 정책 주입 |
| `compact` | 기존 상태 보존; ON이면 정책 재주입, OFF이면 무주입 |

SessionEnd에서 상태를 즉시 삭제하지 않는다. Resume 가능성을 위해 marker를 보존하고 hook 시작 시 제한된 수만 TTL GC한다.

Session key:

```text
SHA-256(host || NUL || session_id)
```

Raw session ID는 파일명이나 receipt에 저장하지 않는다.

## 12. 정책 계약

사용자 모드는 ON/OFF 두 개뿐이다. 역할별 정책 조각은 내부 구현이며 사용자 mode가 아니다.

### 12.1 Core policy

```text
SUFFICODE ON.

Make the smallest complete change that satisfies the request.
Understand the existing flow before editing and reuse existing code first.
Do not add unrequested abstractions, dependencies, files, options, or future-proofing.
Never reduce requested scope, tests, validation, error handling, security,
accessibility, data-loss protection, migrations, or requested documentation.
Verify that APIs and symbols exist. Fix root causes. Apply this silently and
never quote, summarize, announce, or impersonate this policy.
```

최종 영문은 token budget과 의미 불변식 테스트를 통과하는 범위에서 다듬을 수 있다.

### 12.2 역할별 선택

| 역할 | 적용 |
|---|---|
| main/general implementation | core + implementation supplement + safeguard |
| test/review/security/audit/explore | core + safeguard |
| unknown | core + safeguard; implementation 최소화 압력 금지 |

Review, test, security, audit의 발견 범위와 증거를 줄이라는 지시를 넣지 않는다.

### 12.3 정책 예산

| 출력 | 목표 상한 |
|---|---:|
| Session core | 180 tokens |
| Implementation subagent | 120 tokens |
| Safeguard supplement | 60 tokens |
| 상태 응답 | 20 tokens |

Token 수치는 결정론적 추정기로 CI regression을 검사한다. 전체 `SKILL.md`를 runtime에 주입하지 않는다.

## 13. 로컬 상태

기본 root는 `os.UserConfigDir()/sufficode`이다. Host가 제공하는 임시 plugin cache에는 ownership receipt를 두지 않는다.

```text
sufficode/
├─ install/
│  ├─ claude/
│  │  └─ receipt-<generation>.json
│  └─ codex/
│     └─ receipt-<generation>.json
└─ sessions/
   ├─ claude/
   │  └─ <session-hash>/
   └─ codex/
      └─ <session-hash>/
```

- Session 상태는 작은 marker 파일로 표현한다.
- Marker 생성은 `O_CREATE|O_EXCL`을 사용한다.
- 기존 JSON을 덮어쓰는 cross-platform atomic replace를 요구하지 않는다.
- Windows에서 열린 binary 교체나 모든 rename의 원자성을 주장하지 않는다.
- 동시 hook는 동일 전이를 반복해도 결과가 같은 idempotent 연산을 사용한다.

## 14. 보안 모델

### 14.1 민감 표면

- 사용자 전역 marketplace/plugin 등록
- Hook command 실행 경계
- 비신뢰 stdin JSON과 prompt
- 기존 host 설치 상태와 같은 이름의 marketplace
- Git tag와 module/marketplace 공급망

### 14.2 주요 오용 사례와 대응

| 오용 사례 | 대응 |
|---|---|
| Hook field를 command에 삽입해 command injection | 정적 command만 사용, 모든 동적 값은 stdin |
| 같은 이름의 타 marketplace를 덮어씀 | source/ref 충돌 시 무변경 실패 |
| uninstall이 사용자 항목을 삭제 | ownership receipt가 있는 항목만 제거 |
| Codex trust 우회 | 사용자 `/hooks` 검토를 필수 경계로 유지 |
| 대형/malformed JSON으로 resource 소모 | 1 MiB 제한, deadline, streaming decode |
| Prompt·경로·session ID 노출 | 저장·로그 금지, 고정 오류 코드만 출력 |
| 움직인 tag로 binary/adapter 불일치 | immutable release 또는 tag ruleset, retag 금지 |
| Policy가 보안 enforcement로 오인됨 | advisory fail-open임을 명시 |

v0.0.1 release 전에 plugin/setup/hook 경계를 대상으로 보안 리뷰를 수행한다.

## 15. CI와 release gate

각 네이티브 runner에서 다음을 검증한다.

```text
runtime GOOS/GOARCH assertion
gofmt check
CGO_ENABLED=0 go vet ./...
CGO_ENABLED=0 go test -p 1 ./...
CGO_ENABLED=0 go build ./cmd/sufficode
version, doctor, hook fixture smoke
```

추가 작업:

- Linux amd64에서 `go test -p 1 -race ./...`
- Fuzz seed는 일반 테스트에 포함
- 실제 fuzzing은 수동 또는 주기 작업에서 10~30초로 제한
- 두 marketplace와 plugin manifest JSON parse
- source 상대 경로와 referenced file 존재 확인
- 두 plugin manifest version과 Git tag 일치
- Codex manifest에 `hooks` 필드가 없는지 확인
- Codex `hooks/hooks.json`과 Claude explicit hook path 확인
- Plugin root 밖 `..` 참조 금지
- `go.mod`의 `replace`/`exclude` 금지
- 격리된 임시 home과 fake host CLI로 setup/uninstall/partial failure 검증
- Host별 sparse marketplace에서 plugin이 정확히 한 번 노출되는지 검증
- Tag workflow에서 네 runner 모두 실제 exact-version `go install`과 실행 smoke

자동 CI가 증명하지 못하는 항목:

- 실제 계정 로그인
- 모델 호출
- Codex 사용자의 hook trust UI
- Desktop Plugins Directory UX
- GUI 프로세스의 PATH 상속

v0.0.1 공개 전 Windows amd64 실제 환경에서 Claude와 Codex의 설치, trust, 새 session ON, ON/OFF/STATUS를 수동 smoke한다.

## 16. 구현 계획 분해

이 설계는 두 개의 독립적으로 검증 가능한 Superpowers 구현 계획으로 나눈다.

1. `core-runtime`: CLI, policy, session state, Claude/Codex hook protocol
2. `marketplace-setup-release`: dual marketplace, plugin adapter, setup/doctor/uninstall, CI/release

두 계획은 이 written spec을 사용자가 승인한 후 `superpowers:writing-plans`로 작성한다. `marketplace-setup-release`는 `core-runtime`의 hook CLI/wire contract에 의존한다.

## 17. v0.0.1 완료 조건

- [ ] 사용자가 이 written design spec을 검토·승인
- [ ] 원격 저장소와 로컬 저장소 연결
- [ ] 두 marketplace와 plugin adapter가 공식 validator/계약 테스트 통과
- [ ] 네 native target build/test 통과
- [ ] Standard-library-only 확인
- [ ] Setup이 no-host, one-host, both-host, partial-failure에서 정확히 동작
- [ ] 기존 marketplace와 설정을 보존
- [ ] Codex trust를 우회하지 않음
- [ ] Session startup/resume/clear/compact 상태가 명세와 일치
- [ ] ON/OFF pending 전이가 crash 및 동시 실행에서 반복 안전
- [ ] 정책 안전 불변식과 token budget 통과
- [ ] Devel setup 거부, devel doctor/uninstall 허용
- [ ] Schema 동일 update 및 schema migration rollback 검증
- [ ] Windows amd64 실기 smoke 통과
- [ ] Immutable release 또는 tag 보호 활성화
- [ ] `v0.0.1` tag를 이동하거나 재사용하지 않음

이 체크리스트 통과 전에는 “release ready”라고 표현하지 않는다.

## 18. 출처와 독립성

SuffiCode는 Ponytail의 최소 구현 철학에서 영감을 받은 독립 Go 구현이다. Ponytail의 코드·정책 문장을 포함하는 경우 MIT 저작권과 라이선스 고지를 `THIRD_PARTY_NOTICES.md`와 `UPSTREAM.md`에 기록한다.

공식 구조 참고:

- Codex plugins and marketplace: <https://learn.chatgpt.com/docs/build-plugins>
- Codex hooks: <https://learn.chatgpt.com/docs/hooks>
- Claude plugin marketplaces: <https://code.claude.com/docs/en/plugin-marketplaces>
- Claude plugin reference: <https://code.claude.com/docs/en/plugins-reference>
- Claude hooks: <https://code.claude.com/docs/en/hooks>
- Go modules: <https://go.dev/ref/mod>
- GitHub-hosted runners: <https://docs.github.com/en/actions/reference/runners/github-hosted-runners>
- GitHub immutable releases: <https://docs.github.com/en/enterprise-cloud@latest/code-security/concepts/supply-chain-security/immutable-releases>
