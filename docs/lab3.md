# レポート課題3（3日目）：Docker環境の構築

本実験は1回分（4時限分 × 1）を想定している。

実験レポートのファイル名は T3情報工学3_[学生番号]_[氏名].pdf とすること。

## 【基礎知識】Dockerとコンテナ
Dockerは、アプリケーションの実行環境をコンテナとして分離し、同じ構成を再現しやすくする仕組みである。後続の実験でWeb UIやAI関連サービスを動かすため、事前にDockerをセットアップしておく。

## 1. 目的
Dockerをセットアップし、まず単体コンテナの実行を体験したうえで、`docker compose` による複数設定の管理方法を習得する。

## 作業区分ラベル
- `[Webブラウザ(Proxmox)]`: Proxmoxの画面操作
- `[Windows 11 ターミナル]`: Windows 11 の PowerShell 操作
- `[Ubuntuサーバー ターミナル]`: Ubuntu Server 上のコマンド操作

## 2. 実験方法（使用機器と手順）
まず `docker run` で単体コンテナを実行し、その後で `docker compose` による構成管理を体験する。

### 2.1 Docker単体の基本操作を確認する

1. [Ubuntuサーバー ターミナル] Dockerのインストール

```bash
sudo apt update
sudo apt install -y docker.io docker-compose-v2
```

1. [Ubuntuサーバー ターミナル] Dockerサービスの有効化

```bash
sudo systemctl enable --now docker
sudo systemctl status docker
docker --version
```

1. [Ubuntuサーバー ターミナル] 単体コンテナの動作確認

```bash
sudo docker run --rm hello-world
```

1. [Ubuntuサーバー ターミナル] 起動中コンテナと取得済みイメージを確認する

```bash
sudo docker ps -a
sudo docker images
```

### 2.2 Docker Composeで構成管理を体験する

1. [Ubuntuサーバー ターミナル] Composeプラグインが利用できることを確認する

```bash
sudo docker compose version
```

1. [Ubuntuサーバー ターミナル] `docker compose` の動作確認用ディレクトリを作成する

```bash
sudo mkdir -p /opt/compose-demo
cd /opt/compose-demo
```

1. [Ubuntuサーバー ターミナル] `compose.yaml` を作成する

```bash
sudo nano /opt/compose-demo/compose.yaml
```

以下の内容を保存する。

```yaml
services:
  web:
    image: traefik/whoami
    container_name: compose-whoami
    ports:
      - "8080:80"
    restart: unless-stopped
```

1. [Ubuntuサーバー ターミナル] Compose定義を検証して起動する

```bash
cd /opt/compose-demo
sudo docker compose config
sudo docker compose up -d
sudo docker compose ps
```

1. [Ubuntuサーバー ターミナル] 起動したコンテナを検証する

```bash
curl http://localhost:8080
sudo docker compose logs --tail 20
```

`curl http://localhost:8080` の出力には、コンテナ名やIPアドレスなどの情報が表示される。応答本文が返ることを確認する。

1. [Windows 11 ターミナル] クライアントから到達確認を行う

```powershell
Test-NetConnection 192.168.1.10 -Port 8080
```

1. [Windows 11 ターミナル] Webブラウザで `http://192.168.1.10:8080` にアクセスし、whoamiコンテナの応答画面が表示されることを確認する

1. [Ubuntuサーバー ターミナル] Composeで起動したコンテナを停止・削除する

```bash
cd /opt/compose-demo
sudo docker compose down
sudo docker compose ps
```

## 3. 結果と考察
**[Ubuntuサーバー ターミナル] 確認コマンド:**

```bash
docker --version
sudo systemctl status docker
sudo docker run --rm hello-world
sudo docker ps -a
sudo docker images
sudo docker compose version
cd /opt/compose-demo
sudo docker compose config
sudo docker compose up -d
sudo docker compose ps
curl http://localhost:8080
sudo docker compose logs --tail 20
sudo docker compose down
```

**[Windows 11 ターミナル] 確認コマンド:**

```powershell
Test-NetConnection 192.168.1.10 -Port 8080
```

📷 **【エビデンス取得】** 以下の6点をキャプチャし、図表のルールにしたがってレポートにまとめること。

1. `docker --version` と `sudo systemctl status docker` を実行し、Docker本体の導入とサービス起動が確認できる画面
2. `sudo docker run --rm hello-world` を実行し、単体コンテナが正常に起動・終了する画面
3. `sudo docker ps -a` または `sudo docker images` を実行し、Dockerがコンテナやイメージを管理していることがわかる画面
4. `sudo docker compose version` を実行し、Composeプラグインが利用可能であることがわかる画面
5. `sudo docker compose config` と `sudo docker compose ps` を実行し、Compose定義が正しく解釈されて whoami コンテナが起動していることがわかる画面
6. `curl http://localhost:8080`、`Test-NetConnection 192.168.1.10 -Port 8080`、またはブラウザ表示を確認し、Composeで起動した whoami コンテナへサーバー側とクライアント側の両方からアクセスできることがわかる画面

## 4. 課題
1. コンテナ型仮想化とハイパーバイザ型仮想化の違いを説明せよ。
2. コンテナを利用することで、アプリケーション配布や環境再現にどのような利点があるか考察せよ。
3. `compose.yaml` でポート公開や再起動ポリシーを定義しておく利点を説明せよ。

