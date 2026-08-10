---
title: "AWS ALB/WAFの障害切り分けを疑似体験できるハンズオンを作っている話【PathTraceLab】"
emoji: "🧭"
type: "tech"
topics: ["go", "aws", "docker", "traefik"]
published: false
---

## はじめに

**PathTraceLab**（https://path-trace-lab.sorapenguin.dev）は、AWS ALB（ロードバランサー）・WAFの設定を変えながら、403/404/502/503がどこで発生しているかを切り分けるハンズオン型の学習ツールです。

構成（Client→ALB→Application）を確認し、設定（WAFのルール、ターゲットの状態、アプリケーションの応答など）を選んでから仮想テストを実行すると、その設定でどんなHTTPレスポンスが返るかを確認できます。

![設定画面。ApplicationがどのHTTPステータスを返すかを選んで仮想テストを実行できる](/images/pathtracelab/play.png)

## なぜ作ったか

姉妹サービスのRouteLab（CLIコマンド練習）・RunbookLab（Linux/AWSトラブルシュート演習）と役割を分けています。

- RouteLab: 何を**入力するか**（コマンドを実際にタイプする）
- RunbookLab: 何が**壊れてどう直すか**（障害調査→診断→対処）
- PathTraceLab: 設定を変えると**なぜ結果が変わるか**（ALB/WAFの設定とHTTPレスポンスの因果関係）

ALB・WAFはAWS認定資格でも頻出ですが、「WAFのルールを変えたら403になった」「ターゲットグループが空だと503になる」といった因果関係は、実際に手を動かして初めて腹落ちする部分だと思います。実クラウドを契約させずに、この因果関係だけを疑似体験させる、というのが狙いです。

## 名前の話

当初は開発名のまま「GoLab」という名前で進めていましたが、調べてみると同名の著名なGoカンファレンス（イタリア発、Go Contributor Summitも主催する国際イベント）が既にあることが分かり、公開名だけ「PathTraceLab」に改名しました。開発上のモジュール名やフォルダ名は引き続き`golab`のままです。

## 作り方

Go言語の標準ライブラリだけで作っていて、外部ライブラリに頼らないシンプルな構成です。学習シナリオはコード側とは切り離してJSONで管理しているので、コードに手を入れなくてもシナリオを増やしていけます。

現在はAWS ALB/WAFまわりのシナリオを揃えているところで、これからも増やしていく予定です。

## 使ってみてください

**https://path-trace-lab.sorapenguin.dev**

姉妹サービスのRouteLab・RunbookLabと合わせて、よければ覗いてみてください。

---

*Go / Docker / Traefik で構築・VPS本番稼働中*
