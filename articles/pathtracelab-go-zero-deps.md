---
title: "AWS ALB/WAFの障害切り分けを疑似体験できるハンズオンをGo標準ライブラリだけで作っている話【PathTraceLab】"
emoji: "🧭"
type: "tech"
topics: ["go", "aws", "docker", "traefik"]
published: false
---

## はじめに

**PathTraceLab**（https://path-trace-lab.sorapenguin.dev）は、AWS ALB（ロードバランサー）・WAFの設定を変えながら、403/404/502/503がどこで発生しているかを切り分けるハンズオン型の学習ツールです。

先に断っておくと、**まだ開発途中のプロジェクト**です。AWS ALB/WAF関連のシナリオを21本用意していますが、実ブラウザでの最終受入チェックはまだ済んでいません。「今こういうものを作っている」という技術的な話として書きます。

## なぜ作ったか

姉妹サービスのRouteLab（CLIコマンド練習）・RunbookLab（Linux/AWSトラブルシュート演習）と役割を分けています。

- RouteLab: 何を**入力するか**（コマンドを実際にタイプする）
- RunbookLab: 何が**壊れてどう直すか**（障害調査→診断→対処）
- PathTraceLab: 設定を変えると**なぜ結果が変わるか**（ALB/WAFの設定とHTTPレスポンスの因果関係）

ALB・WAFはAWS認定資格でも頻出ですが、「WAFのルールを変えたら403になった」「ターゲットグループが空だと503になる」といった因果関係は、実際に手を動かして初めて腹落ちする部分だと思います。実クラウドを契約させずに、この因果関係だけを疑似体験させる、というのが狙いです。

## Goの標準ライブラリだけで作る

`go.mod`に外部依存が一つもありません。

```
module golab

go 1.22
```

HTTPサーバー・JSONハンドリング・ルーティングまで全部標準ライブラリです。RouteLab（React+TypeScript+Express）・RunbookLab（.NET+EF Core+PostgreSQL）と違い、このプロジェクトはDBもフロントエンドフレームワークも要りません。シナリオ定義（JSON）とロジックだけの軽量ツールなので、あえて依存を増やさない方針にしています。

静的アセット（HTML/CSS/JS・シナリオJSON）は`//go:embed`でバイナリに埋め込んでいるので、成果物はシングルバイナリです。Dockerイメージもマルチステージビルドで`golang:1.22-alpine`→`alpine:3.20`という最小構成にしています。

## シナリオはStage/Experiment方式のJSONで宣言的に定義

学習コンテンツ（Learning Module）は`stages[].experiments[]`という構造のJSONで記述します。

```
Stage（例: 「WAFルールを疑ってみる」）
  └─ Experiment（例: waf_action=block を選んだ場合）
       ├─ 選択肢（Control/Option）
       ├─ 期待される結論
       └─ よくある誤解・試験での判断ポイント
```

このスキーマにはローダー側で`DisallowUnknownFields()`を効かせていて、旧バージョンのフィールド構造やタイポを含むJSONは起動時に弾かれます。コンテンツを増やすときに「動くけど意図と違う」を防ぐための、地味だけど効いているガードです。

現在AWS ALB/WAF関連で21本のLearning Moduleがあり、`go test -count=1 ./...`で12パッケージ全てPASSする状態を保ちながら追加しています。

## コンテナ化で踏んだバグ：ループバック限定バインドとTraefik

もともとこのアプリは「同一ホスト上のリバースプロキシから127.0.0.1経由でアクセスされる」前提で、`LISTEN_ADDR`が未指定のときはループバックアドレスにしかバインドしない設計でした。

Dockerでは、Traefikは同じホストではなく**別コンテナ**なので、Dockerネットワーク経由でしかこのアプリに届きません。ループバック限定バインドのままでは永久に接続できないことになります。

```go
// 修正前: ループバックアドレスのみ許可
func loopbackAddress(raw string) (string, error) { ... }

// 修正後: ループバックに加えて 0.0.0.0 / :: も許容
func loopbackOrUnspecifiedAddress(raw string) (string, error) { ... }
```

デフォルト値は変えず、`LISTEN_ADDR=0.0.0.0`を明示指定したときだけ挙動が変わるようにしたので、既存のbare-metal/systemd運用（同一ホスト上でリバースプロキシと同居する構成）には影響がありません。

## 今は無料で覗けます（発展途上です）

**https://path-trace-lab.sorapenguin.dev**

まだシナリオも少なく、実機ブラウザでの最終確認も済んでいない状態です。「こういうものを作っている」というのが見えるくらいの気持ちで置いています。今後シナリオを増やしながら育てていく予定です。

---

*Go 1.22（標準ライブラリのみ）/ Docker / Traefik で構築・VPS本番稼働中*
