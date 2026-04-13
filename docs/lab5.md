# レポート課題5（5日目）：ローカルLLM導入とAI連携Webサイトの構築

## 【基礎知識】ローカルLLMとAPI
本演習ではクラウド型AIではなく、ローカルLLM環境を構築し、外部（API）からの制御を理解する。APIを経由することで、外部のクライアント端末（Windows等）からネットワークを通じてAIを直接制御することが可能となる。

## 【基礎知識】Webインターフェースとリバースプロキシ
OllamaはAPIだけでも利用できるが、Webインターフェースを導入することで、ブラウザから対話形式で操作しやすくなる。前の演習で導入済みのDockerと、4日目にComposeで起動したNginxコンテナ（`nginx-proxy`）を利用すれば、リバースプロキシとしてOllama関連サービスを公開し、利用者は単一のWeb入口からアクセスできる。

## 【基礎知識】WebサーバーとAI APIのシステム連携
ローカルLLMを起動したうえでWebサーバーとAI APIを連携させ、独自のAIチャットシステムを完成させる。フロントエンドとバックエンドを接続し、利用者の入力をAIへ渡して応答を返すことで、各要素が連携したシステムとして動作することを確認する。

## 1. 目的
Docker導入済みの環境を前提としてローカルLLM環境を構築し、API操作と Docker Compose によるWebインターフェース設定を行ったうえで、AI連携Webサイトを完成させる。

## 作業区分ラベル
- `[Webブラウザ(Proxmox)]`: Proxmoxの画面操作
- `[Windows 11 ターミナル]`: Windows 11 の PowerShell 操作
- `[Ubuntuサーバー ターミナル]`: Ubuntu Server 上のコマンド操作

## 2. 実験方法（使用機器と手順）

1. [Ubuntuサーバー ターミナル] Ollamaインストールとモデル実行

```bash
curl -fsSL https://ollama.com/install.sh | sh
ollama run gemma:2b
```

上記コマンドでOllamaをインストールし、軽量モデル（gemma:2b）を実行する。

1. [Windows 11 ターミナル] API経由の操作

WindowsのPowerShellから、`curl`を使ってUbuntu上のOllama APIにプロンプトを投げる。

1. [Ubuntuサーバー ターミナル] Dockerネットワークと作業用ディレクトリの準備

    4日目にComposeで起動したNginxコンテナ（`nginx-proxy`）と、これから起動するOpen WebUIコンテナを同じDockerネットワークへ参加させる。

    ```bash
    sudo docker network create app-net
    sudo mkdir -p /opt/open-webui
    sudo mkdir -p /opt/nginx/conf.d
    ```

1. [Ubuntuサーバー ターミナル] 4日目のNginx Compose定義を更新する

    Open WebUI へリバースプロキシできるように、4日目に作成した `/opt/nginx/compose.yaml` に設定ディレクトリのマウントと外部ネットワークを追加する。

    ```bash
    sudo nano /opt/nginx/compose.yaml
    ```

    以下の内容に修正する。

    ```yaml
    services:
      nginx:
        image: nginx:stable
        container_name: nginx-proxy
        ports:
          - "80:80"
        volumes:
          - /opt/nginx/html:/usr/share/nginx/html:ro
          - /opt/nginx/conf.d:/etc/nginx/conf.d:ro
          - /opt/nginx/log:/var/log/nginx
        networks:
          - app-net
        restart: unless-stopped
      goaccess:
        image: allinurl/goaccess
        container_name: goaccess-report
        profiles: ["tools"]
        volumes:
          - /opt/nginx/log:/srv/log:ro
          - /opt/nginx/html:/srv/report
        command:
          - --log-format=COMBINED
          - -f
          - /srv/log/access.log
          - -o
          - /srv/report/goaccess-report.html

    networks:
      app-net:
        external: true
    ```

    修正後、設定を反映する。

    ```bash
    cd /opt/nginx
    sudo docker compose config
    sudo docker compose up -d
    sudo docker compose ps
    ```

1. [Ubuntuサーバー ターミナル] Open WebUI 用の Compose 定義を作成する

    Ollamaをブラウザから利用するため、Docker Compose で Open WebUI コンテナを起動する。

    ```bash
    sudo nano /opt/open-webui/compose.yaml
    ```

    以下の内容を保存する。

    ```yaml
    services:
      open-webui:
        image: ghcr.io/open-webui/open-webui:main
        container_name: open-webui
        environment:
          - OLLAMA_BASE_URL=http://host.docker.internal:11434
        extra_hosts:
          - "host.docker.internal:host-gateway"
        volumes:
          - open-webui:/app/backend/data
        networks:
          - app-net
        restart: unless-stopped

    volumes:
      open-webui:

    networks:
      app-net:
        external: true
    ```

1. [Ubuntuサーバー ターミナル] Webインターフェースを Compose で起動する

    ```bash
    cd /opt/open-webui
    sudo docker compose config
    sudo docker compose up -d
    sudo docker compose ps
    sudo docker compose logs --tail 20
    ```

1. [Ubuntuサーバー ターミナル] NginxによるWebインターフェース公開設定

    4日目にComposeで導入したNginxコンテナ（`nginx-proxy`）を利用し、Open WebUIへリバースプロキシする。設定をホスト側へ保存し、Composeで起動中のNginxコンテナへ反映する。

    ```bash
    sudo nano /opt/nginx/conf.d/open-webui.conf
    cd /opt/nginx
    sudo docker compose exec nginx nginx -t
    sudo docker compose exec nginx nginx -s reload
    ```

    設定ファイルでは、`location /` の中で `proxy_pass http://open-webui:8080;` を記述し、`http://[サーバーIP]/` からOpen WebUIへ接続できるようにする。

1. [Ubuntuサーバー ターミナル] フロントエンドとバックエンドの構築

    `index.html` に入力フォームを作成し（Webフロントエンド）、PHPやPython(Flask)等を用い、Webフォームの入力をOllama APIに転送し、結果を表示する「AIチャット画面」を構築する（バックエンド連携）。

## 3. 結果と考察
`[Windows 11 ターミナル]` から以下のコマンドを実行し、API応答を確認する。

```powershell
curl http://[自分のIP]:11434/api/generate -d '{
  "model": "gemma:2b",
  "prompt": "高専について教えて",
  "stream": false
}'
```

`[Ubuntuサーバー ターミナル]` では、以下のコマンドで Compose による起動状態を確認する。

```bash
cd /opt/nginx
sudo docker compose ps
sudo docker compose logs --tail 20
cd /opt/open-webui
sudo docker compose ps
sudo docker compose logs --tail 20
```

その後、`[Windows 11 ターミナル]` のブラウザから `http://[自分のIP]/` または `http://www.t3.metro-cit.internal` へアクセスし、Open WebUIとAI連携Webサイトの動作を確認する。

📷 **【エビデンス取得】** 以下の5点をキャプチャし、システムが正しく連携していることを考察すること。

1. APIテストを実行し、Ollamaから応答が返ってきた画面
2. `cd /opt/nginx && sudo docker compose ps` を実行し、NginxコンテナがComposeで起動していることがわかる画面
3. `cd /opt/open-webui && sudo docker compose ps` を実行し、Open WebUIコンテナがComposeで起動していることがわかる画面
4. ブラウザからOpen WebUIへアクセスし、画面が表示されていることがわかる画面
5. ブラウザ上でAI連携Webサイトが正常に会話応答できている画面

## 4. 課題
1. クラウド型AI（Gemini/ChatGPT等）と比較した、ローカルLLMの利点と欠点は何か。
2. コマンドライン（ターミナル）から直接LLMを実行するのと比較して、APIを経由してLLMを操作できるようにすることには、システム開発上どのようなメリットがあるか考察せよ。
3. Webフロントエンド、バックエンド、AI APIの3要素は、それぞれどのような役割を担い、どのようにデータを受け渡しているか説明せよ。
