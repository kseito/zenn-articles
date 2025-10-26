---
title: "android-mcp-serverでAndroidの動作確認もClaude Codeにお任せする"
emoji: "🤖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["android", "claude", "mcp", "adb"]
published: false
---

## はじめに

Androidアプリ開発において、コーディング後の動作確認は欠かせない工程です。しかし、この動作確認作業をClaude Codeに任せられたら便利だと思いませんか？

本記事では、android-mcp-serverを使ってClaude CodeでAndroid端末の操作・動作確認を自動化する方法と、実際に使ってみて分かった注意点を紹介します。

## android-mcp-serverとは

[android-mcp-server](https://github.com/minhalvp/android-mcp-server)は、ADB(Android Debug Bridge)コマンドをMCP(Model Context Protocol)経由で扱えるようにしたツールです。

MCPは、AIアシスタントが外部ツールやデータソースと連携するための標準プロトコルで、このサーバーを使うことでClaude CodeからAndroid端末を直接操作できるようになります。

主な機能:
- スクリーンショットの取得
- タップ操作
- テキスト入力
- アプリの起動

## セットアップ

android-mcp-serverをClaude Codeに設定する手順は以下の通りです。

### 前提条件

- Python 3.11以上
- [uv](https://github.com/astral-sh/uv)パッケージマネージャー
- ADB(Android Debug Bridge)が使用できる環境

### インストール手順

1. リポジトリをクローン:
```bash
git clone https://github.com/minhalvp/android-mcp-server.git
cd android-mcp-server
```

2. 依存関係をインストール:
```bash
uv python install 3.11
uv sync
```

3. Claudeのコマンドでandroid-mcp-serverを追加:
```bash
claude mcp add android-mcp-server
```

プロンプトに従って以下のように設定します:
- Command: `uv`
- Arguments: `--directory /path/to/android-mcp-server run android-mcp-server`
  - `/path/to/android-mcp-server`は手順1でクローンしたディレクトリのパスに置き換えてください

4. Claude Codeを再起動

これで準備完了です！

## 実際に動作確認してもらう

セットアップが完了したら、Claude Codeに動作確認を依頼できます。
例えば、以下のように依頼します。

```
MCPをつかって動作確認してくれる？Todoの追加・完了の動作確認をお願いします。
```

するとClaude Codeがandroid-mcp-serverを使い動作確認し結果を報告してくれます。
今回の依頼に対する結果報告は下記のようになりました。

```
⏺ 動作確認結果

  TodoListViewModelでのDispatchers.Defaultから適切なディスパッチャーへの修正が完了
  し、動作確認を行いました。

  実施した修正

  1. TodoDao.kt - 全てのデータベース操作メソッドにsuspendを追加
    - findAll(), findBy(), insertOrUpdate(), completeTaskById(),
  completeTaskByTodoistId(), delete()
    - これによりRoomが自動的にDispatchers.IOで実行するようになりました
  1. TodoListViewModel.kt -
  全てのviewModelScope.launchからDispatchers.Defaultを削除
    - initブロック、refreshTodoList(), updateTodo(), deleteTodo(), completeTodo()
    - 不要なimport kotlinx.coroutines.Dispatchersも削除

  動作確認結果

  ✅ Todo追加機能: 正常に動作
  - FABボタンからBottomSheetを開き、"Working_Todo"という名前のTodoを追加
  - データベースに正常に保存され、リストに表示されました

  ✅ Todo完了機能: 正常に動作
  - チェックボックスをタップしてTodoを完了
  - completeTodo()が呼ばれ、Todoがリストから削除されました

  ✅ ビルド: エラーなく成功

```

これにより、**コーディング → 動作確認 → PR作成** までの一連の流れを全てClaude Codeに任せることができます。うまくいけば開発者は実装の依頼とレビューに集中できるようになり開発を効率化できます。

## 使用時の注意点

実際に使ってみて、いくつか注意すべき点がありました。

### 1. トークン使用量の増加

スクリーンショットの取得や操作の度にトークンを消費するため、通常のコーディングタスクと比べてトークン使用量が大幅に増加します。
体感では**1タスクあたりのトークン量が約2倍**になっている印象です。タスクの規模や動作確認の複雑さによってはさらに増える可能性もあります。
Claude Codeの使用量が気になる方は重要な動作確認のみMCPを使用し簡単な確認は手動で行うなど使い分けを検討すると良いでしょう。

### 2. 日本語入力ができない

現時点では、**キーボードの言語切り替え(日本語・英語)を行うことができません**。
動作確認の際は英語で入力できるテストデータを用意する必要があります。

### 3. タップ座標のズレ

たまに*Claude Codeが意図したコンポーネントとは異なる座標を延々とタップし続ける**という問題が発生します。
これは**端末の解像度情報が正しく伝えていない**ことが原因でした。

解決方法:
最初にClaude Codeに「端末の解像度を確認してください」と伝えると、以降は正確な座標でタップできるようになります。

## まとめ

android-mcp-serverをClaude Codeに組み込むことで、Android アプリの開発フローを効率化できます。

**メリット:**
- コーディングから動作確認、PR作成まで一貫してClaude Codeに任せられる
- 繰り返しの動作確認作業を自動化できる
- 開発者は実装とレビューに集中できる

**注意点:**
- トークン使用量が約2倍に増加する
- 日本語入力には対応していない
- 初回は端末解像度の確認が必要

注意点を理解した上で使えば非常に便利なツールです。特に、単純だけど繰り返し実行が必要な動作確認タスクには最適です。ぜひ試してみてください！

## 参考リンク

- [android-mcp-server GitHub](https://github.com/minhalvp/android-mcp-server)
- [Model Context Protocol](https://modelcontextprotocol.io/)
- [Claude Code](https://docs.claude.com/claude-code)
- [RewardedTodo](https://github.com/kseito/RewardedTodo)
