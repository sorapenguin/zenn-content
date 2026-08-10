# zenn-content

[Zenn](https://zenn.dev/sorapenguin) の記事管理用リポジトリ（Zenn CLI + GitHub連携）。

## 構成

- `articles/*.md` — 記事本文（frontmatterで `published`/`published_at` を管理）

## 公開予約の使い方

frontmatterに以下を指定してpushすると、指定日時（JST）に自動公開される。

```yaml
published: true
published_at: 2026-08-20 08:00
```

`published: false` のままなら下書き扱いで、Zenn上には非公開のまま反映される。

## セットアップ手順（初回のみ・Zenn側の設定）

1. https://zenn.dev/dashboard/deploys を開く
2. 「GitHubからのデプロイ」で `sorapenguin/zenn-content` を連携
3. 以後、このリポジトリへのpushが自動的にZennへ反映される
