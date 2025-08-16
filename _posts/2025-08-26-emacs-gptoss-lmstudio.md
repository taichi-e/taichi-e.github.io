---
title: "Emacs から gpt-oss-20b（LM Studio）を使う方法"
description: >-
  emacs 上で LLMと対話できる gptel に、LM Studio で動作している gpt-oss を設定追加するまでの手順です。
author: taichi-e
date: 2025-08-16 12:00:00 +0900
media_subpath: /assets/img/20250806-gpt-oss/
categories: [emacs]
tags: [LLM, emacs, setup]
---

# Emacs から gpt-oss-20b（LM Studio）を使う方法

## 背景

OpenAI Cookbook の手順を参考に、`gpt-oss-20b` を **LM Studio** でローカル環境に立ち上げることができます。LM Studio は **OpenAI API 互換のローカルサーバ** を提供するため、Emacs の [gptel](https://github.com/karthink/gptel) から直接呼び出して利用できます。

この記事では、**Emacs + gptel + LM Studio** を組み合わせて、ChatGPT のように Emacs 上でローカルLLMを使う方法を紹介します。

---

## 手順

### 1. LM Studio 側の準備

1. LM Studio を起動し、モデル（例: `gpt-oss-20b`）をロードする。
2. メニューから **Developer → Local Server** を開き、サーバを起動する。

   * デフォルトのエンドポイント:

     ```
     http://localhost:1234/v1
     ```
   * API キーは不要だが、OpenAI互換の都合で `"lm-studio"` などダミーキーを使用する。

### 2. モデルIDの確認

ターミナルで以下を実行し、モデルIDを確認します。

```bash
curl http://localhost:1234/v1/models
```

例として、`openai/gpt-oss-20b` のようなIDが返ってきます。これを後で gptel の設定に使います。

### 3. Emacs 側の設定

Emacs で gptel を導入します。

```elisp
;; gptel をロード
(require 'gptel)

;; LM Studio の OpenAI互換サーバを登録
(setq gptel-backend
      (gptel-make-openai "LMStudio"
        :protocol "http"
        :host "localhost:1234"
        :endpoint "/v1/chat/completions"
        :stream t
        :key "lm-studio"                 ;; ダミーキーでOK
        :models '("openai/gpt-oss-20b"))) ;; 確認したIDを指定

;; デフォルトモデルを設定
(setq gptel-model "openai/gpt-oss-20b")
```

### 4. 使い方

* 任意のバッファで `M-x gptel` → プロンプトを入力すると、回答がその場に表示されます。
* 選択範囲を修正してもらうなら `M-x gptel-rewrite` が便利です。

---

## 注意点とトラブルシュート

* **レスポンスがない/モデルが見つからない**
  → `curl http://localhost:1234/v1/models` でモデルIDを確認し、gptel の設定を修正。

* **エンドポイント404**
  → LM Studio の場合は `/v1/chat/completions`。Open WebUI などでは `/api/chat/completions` になるため注意。

* **HTTPSエラー**
  → 自己署名証明書で HTTPS を使う場合、`:curl-args '("--insecure")` を追加。

---

## まとめ

* LM Studio は OpenAI互換APIを提供するので、gptel と組み合わせて Emacs から簡単に利用可能。
* `gptel-make-openai` を使って `localhost:1234/v1` に接続し、ダミーAPIキー `"lm-studio"` を指定すれば動作します。
* これにより、Emacs 上でローカルLLMを ChatGPT のように活用できます。
