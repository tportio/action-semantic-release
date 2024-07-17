<h1 align="center" style="border-bottom: none;">📦🚀 semantic-release</h1>
<h3 align="center">프로젝트와 분리해서 사용하는 Github Custom Action</h3>

**[semantic-release](https://github.com/semantic-release/semantic-release)** 를 사용하기 위해서는 npm 설정이 필요하기 때문에, nodejs 이외의 프로젝트에서 사용하려면 추가적인 설정등이 필요해서 custom action으로 만들게 되었습니다.

아래와 같은 특징이 있습니다.
- `main`, `master`, `next`, `release` 브랜치에서만 semantic-release 를 사용해서 버전을 생성하고 release를 합니다.

- 그 외 브랜치에서는 extra 버전 번호를 추가로 부여해서 관리고, prerelease가 됩니다.
> Note: 이 경우에는 마지막 릴리즈 버전에서 extra 버전 번호가 추가됩니다

| prefix | 마지막 릴리즈 버전  | 다음 릴리즈 버전 |
|-|-|-|
| v | v0.0.1 | v0.0.1.1 |
| v | v0.0.1.1 | v0.0.1.2 |
| submodule-v | submodule-v40.24.7 | submodule-v40.24.7.1 |
| submodule-v | submodule-v0.2.53.123 | submodule-v0.2.53.124 |

- 최초 릴리즈에는 v0.0.1 을 기본으로 릴리즈 하게 됩니다.

### 주의사항

**- Protected Branch**

> Branch Protection 설정이 된 브랜치에서는 Pull Request를 통하지 않는 변경은 허용하지 않기 때문에,
> release tag 생성하는데 제약사항이 있다.
>
> 이를 해결하기 위해서 아래 몇가지 방법 중 하나를 선택한다.
>
> 1. 버저닝을 Branch Protection 설정이 없는 브랜치(개발브랜치나 버저닝브랜치 등)에서 관리한다.
> 2. oragnization level PAT(Personal access token)인 `ONDABOT_TOKEN` 을 사용하고 아래 중 하나를 설정한다.
> - `onda-bot` 사용자를 `Collaborators and teams` 에 admin 으로 등록
> - `Branch protection rules` 설정의 `Allow force pushes` -> `Specify who can force push` 에 등록

**- @semantic-release/git 플러그인**

기본적으로 `@semantic-release/git` 플러그인을 사용하기 때문에, protected branch 설정이 되어 있다면,
`actions/checkout@v4` 설정의 `persist-credentials: false` 파라미터를 넣어줘야 한다.

**- GITHUB_TOKEN**

> `GITHUB_TOKEN` 으로 넘겨진 토큰값은 semantic-release 실행시 아래 환경 변수가 별도로 설정되어 있는지 확인 후, 없다면 기본값으로 함께 사용된다.
> - `GH_TOKEN` : gh cli 사용시 필요
> - `NODE_AUTH_TOKEN`: `@semantic-release/npm` 플러그인 사용시 필요

## Usage

**Basic:**

```yaml
steps:
  - name: Checkout
    uses: actions/checkout@v4
    with:
      persist-credentials: false

  - uses: tportio/action-semantic-release@v1
    id: semantic
    env:
      GITHUB_TOKEN: ${{ secrets.ONDABOT_TOKEN }}

  - name: version
    run: echo "${{ steps.localaction.outputs.version }}"

  - name: version tag
    run: echo "${{ steps.localaction.outputs.tag }}"
```


**Version Prefix:**

```yaml
steps:
  - name: Checkout
    uses: actions/checkout@v4
    with:
      persist-credentials: false

  - uses: tportio/action-semantic-release@v1
    id: semantic
    env:
      GITHUB_TOKEN: ${{ secrets.ONDABOT_TOKEN }}
    with:
      prefix: submodule-

  - name: version
    run: echo "${{ steps.localaction.outputs.version }}"

  - name: version tag
    run: echo "${{ steps.localaction.outputs.tag }}"
```

## Customizing

### .releaserc
project root에 `.releaserc` 파일이 존재하면 해당 파일을 사용합니다.

### inputs

The following inputs can be used as `step.with` keys:

| Name       | Type   | Default | Description                                                                   |
|------------|--------|---------|-------------------------------------------------------------------------------|
| `prefix` | String | v        | 버전 번호 앞에 붙이는 문자열 |
| `node-version` | String | 20        | semantic-release 를 실행할 node 버전 |

### outputs

| Name       | Type   | Description  |
|------------|--------|---------|
| `version` | String | 버전 번호 (1.0.0) |
| `tag` | String | prefix가 포함된 버전명 (v1.0.0) |
| `prerelease` | Boolean | prerelease 여부 |
