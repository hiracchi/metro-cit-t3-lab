---

## レポート課題4（5日目）：Ollamaの導入とAPI操作

### 【基礎知識】ローカルLLMとAPI
本演習ではクラウド型AIではなく、ローカルLLM環境を構築し、外部（API）からの制御を理解する。APIを経由することで、外部のクライアント端末（Windows等）からネットワークを通じてAIを直接制御することが可能となる。

### 1. 目的
ローカルLLM環境を構築し、外部（API）からの制御を理解する。

### 2. 実験方法（使用機器と手順）

1. Ollamaインストールとモデル実行

```bash
curl -fsSL https://ollama.com/install.sh | sh
ollama run gemma:2b
```

上記コマンドでOllamaをインストールし、軽量モデル（gemma:2b）を実行する。

2. API経由の操作
WindowsのPowerShellから、`curl`を使ってUbuntu上のOllama APIにプロンプトを投げる。

### 3. 結果と考察
WindowsのPowerShellから以下のコマンドを実行する。

```powershell
curl http://[自分のIP]:11434/api/generate -d '{
  "model": "gemma:2b",
  "prompt": "高専について教えて",
  "stream": false
}'
```

📷 **【エビデンス取得】** 上記のAPIテストを実行し、Ollamaから応答が返ってきた画面をキャプチャしてレポートに記載すること。

### 4. 課題
1. クラウド型AI（Gemini/ChatGPT等）と比較した、ローカルLLMの利点と欠点は何か？
2. コマンドライン（ターミナル）から直接LLMを実行するのと比較して、今回のようにAPIを経由してLLMを操作できるようにすることには、システム開発上どのようなメリットがあるか考察せよ。
3. 1日目の仮想マシン作成時、メモリを4GBと多めに設定した。OllamaのようなローカルLLMを動作させる際、なぜ多くのメモリ（ハードウェアリソース）が必要になるのか考察せよ。

