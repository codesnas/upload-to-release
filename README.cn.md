# upload-to-release

使用 GitHub REST API 将文件上传到 GitHub Releases 的 Action。

[English](README.md) | [简体中文](README.cn.md)

## 使用说明

在 `.github/workflows/*.yml` 工作流脚本中引用此 Action 即可使用：

```yaml
- name: Upload files to Release
  uses: ophub/upload-to-release@main
  with:
    tag: "My Release tags name"
    artifacts: "<path>/release.tar.gz,foo/*.txt"
    allow_updates: true
    replaces_artifacts: true
    make_latest: true
    gh_token: ${{ secrets.GITHUB_TOKEN }}
```

### 包含所有选项的完整示例

```yaml
- name: Upload files to Release
  uses: ophub/upload-to-release@main
  with:
    tag: "My Release tags name"
    artifacts: "<path>/release.tar.gz,foo/*.txt"
    allow_updates: true
    remove_artifacts: false
    replaces_artifacts: true
    upload_timeout: 10
    make_latest: true
    prerelease: false
    draft: false
    name: "My Release title name"
    body: |
      ### Describe your Releases notes
      - More description...
    out_log: false
    gh_token: ${{ secrets.GITHUB_TOKEN }}
```

### 在后续步骤中使用 Action 输出（可选项）

```yaml
- name: Upload files to Release
  id: upload_step
  uses: ophub/upload-to-release@main
  with:
    tag: "My Release tags name"
    artifacts: "<path>/release.tar.gz,foo/*.txt"
    gh_token: ${{ secrets.GITHUB_TOKEN }}

- name: 打印 Release 地址
  run: echo "Release URL: ${{ steps.upload_step.outputs.html_url }}"
```

## 配置说明

可在工作流文件中配置以下选项：

| 选项 | 是否必填 | 默认值 | 说明 |
|------|----------|--------|------|
| `tag` | **必填** | — | 要创建或更新的 Release 标签名称（如 `v1.0.0`）。 |
| `artifacts` | **必填** | — | 要上传的文件路径，支持 glob 通配符和逗号分隔的多路径（如 `dist/*.zip` 或 `dist/*.zip,out/*.tar.gz`）。 |
| `gh_token` | **必填** | — | 用于 API 认证的 [GITHUB_TOKEN](https://docs.github.com/zh/actions/security-guides/automatic-token-authentication)，需要 `contents: write` 权限。 |
| `repo` | 可选 | 当前仓库 | 目标仓库，格式为 `<owner>/<repo>`，默认为运行工作流的仓库。 |
| `allow_updates` | 可选 | `true` | 当指定标签的 Release 已存在时，是否更新其元数据（名称、正文、标志位）。设为 `false` 则跳过已有 Release 的元数据更新。 |
| `remove_artifacts` | 可选 | `false` | 上传前是否删除 Release 中的**所有**现有资产文件。优先级高于 `replaces_artifacts`。 |
| `replaces_artifacts` | 可选 | `true` | 是否替换同名的已有资产文件。设为 `false` 时，上传同名文件时将会跳过，不进行替换。 |
| `upload_timeout` | 可选 | `10` | 单个文件上传超时时间，单位为**分钟**。超时后自动放弃当前文件并继续上传下一个。设为 `0` 表示禁用单文件最大时限。注意：即使设为 `0`，防卡死的速度守卫仍然有效——若上传速度连续 60 秒低于 1 KB/s，仍会自动放弃该文件。 |
| `make_latest` | 可选 | `true` | 是否将此 Release 标记为最新版本。可选值：`true` / `false` / `legacy`（由发布日期和语义化版本号决定）。 |
| `prerelease` | 可选 | `false` | 是否将此 Release 标记为预发布版本。 |
| `draft` | 可选 | `false` | 是否将此 Release 标记为草稿。 |
| `name` | 可选 | `""` | Release 的显示标题名称，留空时自动使用标签名。 |
| `body` | 可选 | `""` | Release 的 Markdown 正文内容。同时设置 `body_file` 时，此项被覆盖。 |
| `body_file` | 可选 | `""` | Release 正文内容的 Markdown 文件路径，优先级高于 `body`。 |
| `out_log` | 可选 | `false` | 是否输出每个步骤的详细 JSON 日志，便于调试。 |

## 输出参数

| 输出 | 说明 |
|------|------|
| `release_id` | 创建或更新的 Release 的数字 ID。 |
| `html_url` | Release 页面的 HTML 地址（如 `https://github.com/owner/repo/releases/tag/v1.0.0`）。 |
| `upload_url` | Release 资产上传 URL（可用于自定义上传步骤）。 |
| `assets` | JSON 对象，将每个已上传文件名映射到其下载地址（如 `{"file.zip":"https://...","image.img.gz":"https://..."}`）。 |

## 权限配置

工作流任务需具备 **`contents: write`** 权限：

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: ophub/upload-to-release@main
        with:
          tag: v1.0.0
          artifacts: dist/*
          gh_token: ${{ secrets.GITHUB_TOKEN }}
```

## 上传进度与日志

与其他上传 Release 的 Action 不同，本 Action 会为每个文件实时打印详细进度，例如：

```text
[ STEPS ] Expanding artifact patterns...
[ INFO  ] Total files to upload: [ 5 ]
[ INFO  ] ────────────────────────────────────────────────────────────────────────
[ INFO  ]    1/5     1.23 GiB   firmware-arm64.img.gz
[ INFO  ]    2/5    45.67 MiB   firmware-x86.img.gz
[ INFO  ]    3/5     2.10 MiB   checksums.sha256
[ INFO  ]    4/5    12.34 KiB   release-notes.md
[ INFO  ]    5/5   890.00 B     version.txt
[ INFO  ] ────────────────────────────────────────────────────────────────────────
[ INFO  ] Total: [ 5 ] files,  1.28 GiB
[ INFO  ] ────────────────────────────────────────────────────────────────────────

[ STEPS ] Starting upload of [ 5 ] file(s) to release [ 123456 ]...
[ FILE ] ┌─ (1/5) Uploading: [ firmware-arm64.img.gz ]
[ SIZE ] │  (1/5) Size: 1.23 GiB  MIME: application/gzip  timeout=10min
[ DONE ] │  (1/5) Upload completed in 87s: [ firmware-arm64.img.gz ]
[ DONE ] └─ (1/5) Download URL: [ https://github.com/owner/repo/releases/download/v1.0.0/firmware-arm64.img.gz ]

...
[ SUCCESS ] Upload summary: [ 5 ] total, [ 5 ] succeeded, [ 0 ] failed, [ 0 ] skipped.
```

上传开始前还会打印编号文件清单，方便提前确认上传队列中的所有文件。

## 文件完整性校验

全部文件上传完成后，Action 会自动使用 **SHA-256** 对每个成功上传的文件进行完整性校验：

1. GitHub Releases API 在使用 2022-11-28 版本上传资产时，会返回 `digest` 字段（格式为 `sha256:<hex>`），校验时直接使用该字段，**无需额外下载文件**。
2. 若 `digest` 字段缺失（旧版本 Release），则**跳过该文件的校验**，不会执行额外下载——校验仅在 API 返回哈希值时进行。

校验结果以通过/失败表格形式输出。**校验失败不影响整体步骤的成功状态**——只要有文件成功上传，步骤就不会失败。

## 注意事项

- 若指定的 Tag 在仓库中尚不存在，GitHub 将在创建 Release 时自动以默认分支的当前提交为基础创建该 Tag。
- `remove_artifacts: true` 会在上传前删除**所有**现有资产文件，请谨慎使用。
- 当 `replaces_artifacts` 为 `true` 且已存在同名文件时，系统会先删除旧资产，再重新上传。
- 同时提供 `body_file` 和 `body` 时，`body_file` 优先生效。
- 本 Action 会自动检测并通过 `apt-get` 安装缺失的依赖（`jq`、`curl`、`bc`、`file`）。
- 若某个文件上传卡住（连续 60 秒无数据传输，或超过 `upload_timeout` 设定的时限），上传任务会自动放弃当前文件并继续处理队列中的下一个文件。
- 将 `upload_timeout` 设为 `0` 仅禁用单文件最大时限，防卡死速度守卫仍然有效——上传速度连续 60 秒低于 1 KB/s 时仍会自动放弃。

## 相关链接

- [GitHub REST API – Releases](https://docs.github.com/zh/rest/releases/releases)
- [GitHub REST API – Release Assets](https://docs.github.com/zh/rest/releases/assets)
- [delete-releases-workflows](https://github.com/ophub/delete-releases-workflows)
- [amlogic-s9xxx-armbian](https://github.com/ophub/amlogic-s9xxx-armbian)
- [amlogic-s9xxx-openwrt](https://github.com/ophub/amlogic-s9xxx-openwrt)
- [luci-app-amlogic](https://github.com/ophub/luci-app-amlogic)
- [fnnas](https://github.com/ophub/fnnas)
- [kernel](https://github.com/ophub/kernel)
- [u-boot](https://github.com/ophub/u-boot)
- [firmware](https://github.com/ophub/firmware)

## 许可协议

upload-to-release © OPHUB is licensed under [GPL-2.0](https://github.com/ophub/upload-to-release/blob/main/LICENSE).
