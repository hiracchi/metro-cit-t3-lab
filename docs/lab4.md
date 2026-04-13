# レポート課題4（4日目）：Webサーバー構築とインフラ自動化

## 【基礎知識】Webサーバーと自動化
Webサーバーは、ブラウザからの要求に応じてHTMLなどのコンテンツを配信するサーバー機能である。手動でのサーバー構築作業はミスを誘発しやすいため、シェルスクリプトを用いた自動化が現代のインフラ構築では重要視されている。

本演習では、前の演習で導入したDockerを利用し、Nginxをコンテナとして運用する。

DNS（BIND9）の詳細な構築手順と考察項目は、補足資料の [docs/appendix2.md](docs/appendix2.md) を参照すること。

## 1. 目的
Webサーバー（Nginx）を構築し、HTML配信の動作を確認する。あわせてAI（Gemini等）を活用したトラブルシューティングを体験し、シェルスクリプトによる自動化を行う。

## 作業区分ラベル
- `[Webブラウザ(Proxmox)]`: Proxmoxの画面操作
- `[Windows 11 ターミナル]`: Windows 11 の PowerShell 操作
- `[Ubuntuサーバー ターミナル]`: Ubuntu Server 上のコマンド操作

## 2. 実験方法（使用機器と手順）
1. [Ubuntuサーバー ターミナル] Nginx用ディレクトリと Compose 定義ファイルを作成する

```bash
sudo mkdir -p /opt/nginx/html
sudo nano /opt/nginx/compose.yaml
```

    以下の内容を保存する。

```yaml
services:
  nginx:
    image: nginx:stable
    container_name: nginx-proxy
    ports:
      - "80:80"
    volumes:
      - /opt/nginx/html:/usr/share/nginx/html:ro
    restart: unless-stopped
```

1. [Ubuntuサーバー ターミナル] Nginxコンテナを Compose で起動し、初期確認を行う

```bash
cd /opt/nginx
sudo docker compose config
sudo docker compose up -d
sudo docker compose ps
```

起動後、`[Ubuntuサーバー ターミナル]` で `curl` を実行するか、`[Windows 11 ターミナル]` のブラウザから表示を確認する。

1. [Ubuntuサーバー ターミナル] HTMLファイルの作成

    Docker上のNginxから配信するため、ホスト側の `/opt/nginx/html/index.html` を作成する。簡単な見出しと本文を記述し、ブラウザから表示できるようにする。

    ```bash
    sudo nano /opt/nginx/html/index.html
    ```

    例:

    ```html
    <!doctype html>
    <html lang="ja">
    <head>
      <meta charset="utf-8">
      <title>T3 Lab Web Server</title>
    </head>
    <body>
      <h1>T3 Lab Web Server</h1>
      <p>Nginx on Docker is running.</p>
    </body>
    </html>
    ```

1. [Windows 11 ターミナル] ブラウザからの表示確認

    Windows 11 クライアントのブラウザから `http://www.t3.metro-cit.internal` または `http://[サーバーIP]/` にアクセスし、作成したHTMLページが表示されることを確認する。

## 3. 結果と考察
**[Ubuntuサーバー ターミナル] 確認コマンド:**

```bash
curl http://localhost
cd /opt/nginx
sudo docker compose ps
sudo docker compose logs --tail 20
```

📷 **【エビデンス取得】** 以下の3点をキャプチャし、図表のルールにしたがってレポートにまとめること。

1. `curl http://localhost` またはWindows 11 クライアントのブラウザで、作成したHTMLページが表示される画面
2. `cd /opt/nginx && sudo docker compose ps` を実行し、Composeで管理されている Nginx コンテナが起動していることがわかる画面
3. `cd /opt/nginx && sudo docker compose logs --tail 20` を実行し、Nginxコンテナログが確認できる画面

## 4. 課題
1. Webサーバーがブラウザからの要求に応じてコンテンツを返す仕組みを説明せよ。
2. Nginxの設定不備やコンテンツ配置ミスを調査する際、`docker compose ps` や `docker compose logs` の確認がなぜ有効なのか説明せよ。
3. ホスト側で作成したHTMLファイルをDockerコンテナにマウントして配信する方式には、どのような利点があるか説明せよ。

