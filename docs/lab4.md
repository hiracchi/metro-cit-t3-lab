# レポート課題4（4,5日目）：Webサーバー構築とインフラ自動化

本実験は1回分（4時限分 × 2）を想定している。

実験レポートのファイル名は T3情報工学4_[学生番号]_[氏名].pdf とすること。

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
### 2.1 第1段階：Nginxのみを実行・確認する

1. [Ubuntuサーバー ターミナル] Nginx用ディレクトリと Compose 定義ファイルを作成する

```bash
sudo mkdir -p /opt/nginx/html
sudo mkdir -p /opt/nginx/log
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
      - /opt/nginx/log:/var/log/nginx
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

### 2.2 第2段階：Nginx+GoAccessを実行・確認する

1. [Ubuntuサーバー ターミナル] GoAccessを Docker Compose で実行し、アクセスログを可視化

  第1段階で生成されたNginxアクセスログ（`/opt/nginx/log/access.log`）を `goaccess` サービスで解析し、HTMLレポートを生成する。

  ```bash
  cd /opt/nginx
  sudo docker compose run --rm goaccess
  ```

1. [Windows 11 ターミナル] GoAccessレポートの表示確認

  ブラウザで `http://www.t3.metro-cit.internal/goaccess-report.html` または `http://[サーバーIP]/goaccess-report.html` にアクセスし、アクセス解析レポートが表示されることを確認する。

## 3. 結果と考察
**[Ubuntuサーバー ターミナル] 第1段階（Nginxのみ）確認コマンド:**

```bash
curl http://localhost
cd /opt/nginx
sudo docker compose ps
sudo docker compose logs --tail 20
```

**[Ubuntuサーバー ターミナル] 第2段階（Nginx+GoAccess）確認コマンド:**

```bash
cd /opt/nginx
sudo ls -l /opt/nginx/log/access.log
sudo docker compose run --rm goaccess
```

📷 **【エビデンス取得】** 以下の5点をキャプチャし、図表のルールにしたがってレポートにまとめること。

1. 第1段階として `curl http://localhost` またはWindows 11 クライアントのブラウザで、作成したHTMLページが表示される画面
2. 第1段階として `cd /opt/nginx && sudo docker compose ps` を実行し、Composeで管理されている Nginx コンテナが起動していることがわかる画面
3. 第1段階として `cd /opt/nginx && sudo docker compose logs --tail 20` を実行し、Nginxコンテナログが確認できる画面
4. 第2段階として `sudo ls -l /opt/nginx/log/access.log` を実行し、アクセスログがホスト側に保存されていることがわかる画面
5. 第2段階として `http://[サーバーIP]/goaccess-report.html`（または `http://www.t3.metro-cit.internal/goaccess-report.html`）へアクセスし、GoAccessレポートが表示される画面

## 4. 課題
1. Webサーバーがブラウザからの要求に応じてコンテンツを返す仕組みを説明せよ。
2. ホスト側で作成したHTMLファイルをDockerコンテナにマウントして配信する方式には、どのような利点があるか説明せよ。
3. アクセスログをGoAccessで可視化することで、運用監視や障害調査にどのような利点があるか説明せよ。

