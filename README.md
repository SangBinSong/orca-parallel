# Orca Parallel

Codex와 Antigravity에서 명시적 병렬 작업 요청을 처리하는 지침 전용 플러그인입니다. 독립 작업별로 Gemini 4 Hydrogen의 `low`, `medium`, `high`, `xhigh` 중 하나를 고르고, 실행과 결과 수집은 Orca의 기존 `orchestration`을 사용합니다.

## 요구 사항

- Orca에서 코디네이터로 실행 중인 Codex 또는 Antigravity 세션
- 해당 세션에서 실행 가능한 Orca CLI 및 `orca-cli`, `orchestration` 가이드
- 워커 실행 환경에 설치된 Antigravity CLI(`agy`)와 사용할 Hydrogen 모델에 접근 가능한 계정

## 플러그인 등록

[GitHub 저장소](https://github.com/SangBinSong/orca-parallel)는 공개되어 있습니다. 아래 명령으로 GitHub 로그인 없이 소스를 받을 수 있습니다.

### Codex

GitHub 저장소를 마켓플레이스로 등록하고 플러그인을 설치합니다.

```sh
codex plugin marketplace add SangBinSong/orca-parallel
codex plugin add orca-parallel@orca-parallel-local
codex plugin list --marketplace orca-parallel-local
```

로컬에 복제한 프로젝트를 사용하려면 프로젝트 루트에서 `codex plugin marketplace add .`로 등록한 뒤 같은 `codex plugin add` 명령을 실행합니다. [Codex 마켓플레이스 등록 문서](https://developers.openai.com/plugins/build/plugins#add-a-marketplace-from-the-cli)

새 Codex CLI 세션에서 `/skills`로 `orca-parallel` 스킬을 확인합니다. 데스크톱 앱에서는 마켓플레이스 등록 후 앱을 다시 시작하고 **Plugins**에서 `orca-parallel-local`을 선택해 설치할 수 있습니다. [Codex 로컬 플러그인 문서](https://developers.openai.com/plugins/build/plugins#install-a-local-plugin-manually)

### Antigravity

저장소를 복제한 뒤 Antigravity 매니페스트가 들어 있는 `plugins/orca-parallel` 폴더를 설치합니다.

```sh
git clone https://github.com/SangBinSong/orca-parallel.git
cd orca-parallel
agy plugin install ./plugins/orca-parallel
agy plugin list
```

이미 복제했다면 프로젝트 루트에서 `agy plugin install`부터 실행합니다. Antigravity CLI 대화 화면에서는 `/plugin`의 로컬 폴더 설치 메뉴에서 같은 폴더를 선택할 수 있습니다. 새 세션에서 `/orca-parallel`을 스킬 목록에서 확인합니다. [Antigravity 플러그인 문서](https://antigravity.google/docs/plugins?tab=cli)

## 사용

> 로그 분석과 독립 테스트 점검을 병렬로 진행해.

> 서로 독립적인 작업으로 나누고 최대 2개 세션으로 병렬 진행해.

동시 상한의 기본값은 Orca Run당 **10개**입니다. 요청에 적은 상한이 우선하며, `1개`를 지정하면 위임한 작업이 한 번에 하나씩 실행됩니다. 완료 후에도 열려 있는 워커 세션은 자리를 차지합니다. Orca가 세션 해제를 확인하면 대기 중인 작업을 시작합니다. 다른 Run과 수동으로 연 세션은 각자의 수명주기를 따릅니다. [Orca 워커 문서](https://orca.dev/docs/cli/orchestration)

## 확인 방법

```sh
agy plugin validate ./plugins/orca-parallel --strict --json
```

각 호스트에서 명시적 병렬 요청과 기술 용어로만 쓰인 “병렬 실행”을 구분하는지 확인합니다. 실제 실행은 읽기 전용 작업 두 개와 상한 `1개`로 점검할 수 있습니다. Orca 실행 기록에서 첫 워커 세션이 해제된 뒤에 두 번째가 시작되는지, 지정한 모델과 완료 결과가 반환되는지 확인합니다.
