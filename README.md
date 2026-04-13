# metro-cit-t3-lab

このリポジトリは、情報通信工学実験実習 II（情報工学）の実験指導書を管理するためのものである。Windows 11 クライアントから Proxmox 上の Ubuntu Server 24.04 仮想マシンを操作し、初期設定、ファイアウォール、SSH 鍵認証、Docker、Docker Compose、Web サーバー、ローカル LLM、AI 連携 Web サイトの構築までを段階的に学ぶ。

公開されている本文は MkDocs で管理しており、演習本体は 5 本のレポート課題と 2 本の付録で構成される。

## ドキュメント構成

- [docs/lab1.md](docs/lab1.md): 仮想化基盤への接続、Ubuntu Server の導入、初期設定、更新、サービス管理、時刻同期、ログ確認、hosts 設定
- [docs/lab2.md](docs/lab2.md): UFW によるファイアウォール設定、Windows 11 からの確認、SSH 鍵認証、SCP 転送
- [docs/lab3.md](docs/lab3.md): Docker の基本操作、単体コンテナの実行、Docker Compose の体験と検証
- [docs/lab4.md](docs/lab4.md): Docker Compose を用いた Nginx Web サーバー構築、HTML 配信、動作確認
- [docs/lab5.md](docs/lab5.md): Ollama、Open WebUI、Nginx リバースプロキシ、AI 連携 Web サイトの構築
- [docs/appendix.md](docs/appendix.md): Linux 基本コマンド、ファイルとディレクトリ、権限、nano と vim、Windows/macOS との差異
- [docs/appendix2.md](docs/appendix2.md): BIND9 による DNS 構築の補足資料

## 演習の流れ

1. 1日目は Proxmox 上で Ubuntu Server を導入し、ネットワーク設定、OS 更新、サービス管理、ログ確認までを行う。
2. 2日目は UFW と SSH を扱い、安全に遠隔操作できる状態を作る。
3. 3日目は Docker を単体で体験した後、Docker Compose で構成管理を学ぶ。
4. 4日目は Docker Compose で Nginx を運用し、HTML を配信する。
5. 5日目は Ollama と Open WebUI を導入し、Nginx 経由で AI 利用環境を構築する。

## 実験環境の前提

- クライアント端末は Windows 11 を用いる。
- 仮想化基盤は Proxmox を用いる。
- サーバー OS は Ubuntu Server 24.04 を用いる。
- ドメインは t3.metro-cit.internal を用いる。
- Proxmox 上の VM ID は `100 + 学生番号2桁` を用いる。
- サーバー IP アドレスは `192.168.100.[100 + 学生番号2桁]` を用いる。
- サブネットマスクは `255.255.255.0` または `/24` を用いる。
- ホスト名は `pc[学生番号2桁]` を用いる。
- NTP サーバーは `192.168.100.1` を用いる。
- Docker は 3 日目で導入し、4 日目以降は Docker Compose を利用する前提で記述している。

## 作業区分の考え方

本文中では、作業環境の取り違えを防ぐため、以下のラベルを使っている。

- `[Webブラウザ(Proxmox)]`: Proxmox の GUI 操作
- `[Windows 11 ターミナル]`: Windows 11 の PowerShell 操作
- `[Ubuntuサーバー ターミナル]`: Ubuntu Server 上のコマンド操作

## 主な技術要素

- Ubuntu Server 24.04
- Proxmox
- Netplan
- systemd / systemd-timesyncd / journalctl / dmesg
- sudo / SSH / SCP
- ufw
- Docker / Docker Compose
- Nginx
- Ollama
- Open WebUI
- HTTP API
- BIND9（補足資料）

## AIに設定しておくべき内容

本実験では、AI を単なる文章生成ツールではなく、Linux サーバー構築、設定確認、トラブルシューティング、レポート補助のための支援役として利用する。AI に指示する際は、以下の前提を共有しておくこと。

### AIへの前提条件

- 対象科目は高専の情報通信工学実験実習である。
- 受講者は Windows 11 クライアントから Proxmox 上の Ubuntu Server 24.04 仮想マシンを操作する。
- 実験は lab1 から lab5 まで段階的に進み、Docker Compose を含む構成管理や AI 連携まで扱う。
- 利用する主な技術は Netplan、ufw、SSH、Docker、Docker Compose、Nginx、Ollama、Open WebUI、HTTP API、BIND9 である。
- ドメインは t3.metro-cit.internal を用いる。
- ホスト名は pc[学生番号2桁] を用いる。
- サーバー IP アドレスは 192.168.100.[100 + 学生番号2桁] を用いる。

### AIへの出力方針

- 手順を示す場合は、コマンド、設定ファイル、確認方法を分けて示す。
- コマンドは Ubuntu Server 24.04 または Windows 11 PowerShell でそのまま実行できる形を優先する。
- 設定変更を提案する場合は、変更前に確認すべきファイル名やサービス名も併記する。
- エラー対応では、まず原因候補、次に確認コマンド、最後に修正案を示す。
- レポート支援では、完成済みの考察文をそのまま出すのではなく、要点、論点、書くべき観点を整理して示す。
- 環境ラベルの違いを意識し、Windows 側の操作と Ubuntu 側の操作を混在させない。

### AIへの制約

- 実験環境と無関係な過剰な構成変更は行わない。
- root を前提にした危険な操作や、検証なしの設定上書きは避ける。
- 学生が理解しにくい抽象的な説明だけの回答を避ける。
- 課題の答えを丸写しできる完成レポートの生成を避ける。

### AIに期待する支援

- コマンドの意味の説明
- ログの読み取り補助
- ufw、SSH、Docker、Docker Compose、Nginx、Ollama、BIND9 などの設定ミスの切り分け
- シェルスクリプトや設定ファイルのたたき台作成
- レポート課題に対する観点整理

## ドキュメント確認方法

MkDocs を用いてローカル確認する場合は、以下を利用する。

```bash
mkdocs serve
```

または、リポジトリ付属のスクリプトを利用する。

```bash
./serve.sh
```

