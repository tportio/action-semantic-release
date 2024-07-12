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

**Basic:**

```yaml
steps:
  - name: Checkout
    uses: actions/checkout@v4

  - uses: jadewon/semantic-release@v1
    id: semantic
    env:
      GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}

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

  - uses: jadewon/semantic-release@v1
    id: semantic
    env:
      GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    with:
      prefix: submodule-

  - name: version
    run: echo "${{ steps.localaction.outputs.version }}"

  - name: version tag
    run: echo "${{ steps.localaction.outputs.tag }}"
```


## 주의사항
자체적으로 github cli `gh` 를 사용해서 release를 하기 때문에,
`GH_TOKEN` 환경변수를 꼭 설정해주세요.

## Customizing

### .releaserc
project root에 `.releaserc` 파일이 존재하면 해당 파일을 사용합니다.

사용된 plugins 관련해서 필요한 환경변수도 함께 설정해주셔야 합니다.

##### 예
- GITHUB_TOKEN
- NODE_AUTH_TOKEN

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
