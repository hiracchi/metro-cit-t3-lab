# レポート課題1（1日目）：仮想化基盤への接続、OSインストール、初期設定

## 【基礎知識】OSの種類とサーバー用OS
コンピュータのソフトウェアは大きく分けて基本ソフトウェア（OS）、アプリケーション、ミドルウェアの3種類がある。OSにはクライアント用とサーバー用があり、本演習ではサーバー用OSとしてLinux系の「Ubuntu Server」を使用する。サーバー用OSはGUI（グラフィカルな操作画面）を持たず、コマンド入力によるCUIで管理するのが一般的である。

## 【基礎知識】IPアドレスとリモート管理の基本
情報機器をネットワークに参加させるためには、ホスト名、IPアドレスとネットマスク、デフォルトゲートウェイのパラメータが必要である。IPアドレスのうち、全世界で通信可能なものをグローバルアドレス、組織内（LAN）で利用されるものをプライベートアドレスと呼ぶ。

## 【基礎知識】OS更新とQEMU Guest Agent
OSやソフトウェアは、初期インストール直後でも更新が提供されていることが多い。`apt update` は利用可能なパッケージ情報を更新し、`apt upgrade` はインストール済みパッケージを新しい版へ更新する。加えて、Proxmox上の仮想マシンではQEMU Guest Agentを有効にし、ゲストOS側にも `qemu-guest-agent` を導入することで、IPアドレスの取得や安全なシャットダウンなどの連携機能を利用できる。

## 【基礎知識】サービス管理とsudo
Ubuntuでは、各種サーバー機能は「サービス（daemon）」として動作し、`systemctl` で状態確認、起動、停止、再起動、自動起動設定を管理する。加えて、管理者権限が必要な操作は `sudo` を通じて実行する。`sudo` を適切に設定することで、常時rootで作業せずに安全に管理作業を行える。

時刻同期も重要なサービス管理対象であり、Ubuntuでは `systemd-timesyncd` がNTPクライアントとして動作する。サーバー時刻がずれると、ログ解析、証明書検証、認証処理などで問題が発生するため、初期設定時にNTP同期状態を確認しておくことが望ましい。

## 1. 目的
Proxmoxの基本操作を学び、Linuxサーバー（Ubuntu Server）を最小構成で立ち上げたうえで、サービス管理の基本操作、サーバーログ確認、systemd-timesyncdによるNTPクライアント設定、システムの最新化、sudo設定の確認、QEMU Guest Agentの導入、ネットワーク設定、hostsファイル設定を行う。

## 作業区分ラベル
- `[Webブラウザ(Proxmox)]`: Proxmoxの画面操作
- `[Windows 11 ターミナル]`: Windows 11 の PowerShell 操作
- `[Ubuntuサーバー ターミナル]`: Ubuntu Server 上のコマンド操作

## 2. 実験方法（使用機器と手順）
1. [Webブラウザ(Proxmox)] Proxmoxへのログイン

    Windows 11クライアントのWebブラウザ（Chrome）から指定されたURLへアクセスし、配布されたユーザー名とパスワードでログインする。

2. [Webブラウザ(Proxmox)] VM（仮想マシン）の作成

    指定されたリソースプール内で「Create VM」を選択し、以下のように設定する。

    - **OS**: Ubuntu 24.04 ISOを選択。
    - **System**: QEMU Agentをチェック。
    - **Disks**: 32GB程度。
    - **CPU/Memory**: 2コア / 4GB

3. [Webブラウザ(Proxmox)] OSインストール

    - **Language**: English推奨。
    - **Type**: `Ubuntu Server (minimized)`（最小構成のサーバー版）を選択。
    - **Network**: いったんDHCP（IPアドレスの自動割り当て）で進める。

4. [Ubuntuサーバー ターミナル] サービスの一覧確認と管理方法の確認

    サーバーで動作しているサービスの一覧と、管理コマンドを確認する。systemd-timesyncdサービスを例に、状態確認と再起動を試す。

    ```bash
    systemctl list-units --type=service --state=running
    systemctl status systemd-timesyncd
    sudo systemctl restart systemd-timesyncd
    systemctl is-enabled systemd-timesyncd
    ```

5. [Ubuntuサーバー ターミナル] サーバーログの確認

    障害調査の基本として、`journalctl` と `dmesg` によるログ確認方法を習得する。サービス全体の直近ログ、特定サービス（systemd-timesyncd）のログ、起動ログ（カーネルメッセージ）を確認する。

    ```bash
    journalctl -n 30
    journalctl -u systemd-timesyncd -n 30
    journalctl -p warning -n 30
    dmesg | tail -n 30
    ```

6. [Ubuntuサーバー ターミナル] systemd-timesyncd（NTPクライアント）の設定

    サーバー時刻を自動同期するため、`systemd-timesyncd` の状態確認とNTP有効化を行う。必要に応じてNTPサーバー設定を確認する。

    ```bash
    timedatectl status
    sudo timedatectl set-ntp true
    systemctl status systemd-timesyncd
    timedatectl show-timesync --all
    ```

7. [Ubuntuサーバー ターミナル] システムの最新化

    OSインストール完了後にログインし、パッケージ一覧とインストール済みソフトウェアを更新する。

    ```bash
    sudo apt update
    sudo apt upgrade -y
    sudo apt autoremove -y
    ```

8. [Ubuntuサーバー ターミナル] sudoの設定確認

    管理作業に必要な `sudo` が使えることを確認し、必要に応じて sudoers 設定を確認する。

    ```bash
    sudo -l
    id
    sudo visudo
    ```

    `sudo visudo` では `%sudo ALL=(ALL:ALL) ALL` のような設定を確認する。

9. [Webブラウザ(Proxmox)] QEMU Guest Agentの有効化

    ProxmoxのVM設定画面で、対象仮想マシンの `Options` から `QEMU Guest Agent` を有効化する。

10. [Ubuntuサーバー ターミナル] QEMU Guest Agentのインストール

    Ubuntu側で以下のコマンドを実行してGuest Agentをインストールし、起動状態を確認する。

    ```bash
    sudo apt install -y qemu-guest-agent
    sudo systemctl enable --now qemu-guest-agent
    systemctl status qemu-guest-agent
    ```

11. [Ubuntuサーバー ターミナル] ネットワーク状態の確認

    OSインストール完了後、コンソールでログインし、割り当てられたIPアドレスを確認する。

    ```bash
    ip a
    ```

12. [Ubuntuサーバー ターミナル] 設定ファイルのバックアップ

    固定IP設定を行う前に、現在のNetplan設定ファイルをバックアップする。

    ```bash
    sudo cp /etc/netplan/00-installer-config.yaml /etc/netplan/00-installer-config.yaml.bak
    ```

13. [Ubuntuサーバー ターミナル] 固定IP設定

    `vi` や `nano` エディタで設定ファイルを編集し、指定されたIPアドレス、ゲートウェイ、DNS（8.8.8.8等）を設定する。

14. [Windows 11 ターミナル] hostsファイル設定

    今後の演習でホスト名アクセスを行うため、Windows の `C:\Windows\System32\drivers\etc\hosts` にサーバーの固定IPとホスト名の対応を追加する。

    例:

    ```text
    192.168.1.10    www.t3.metro-cit.internal
    ```

    設定後、`ping www.t3.metro-cit.internal` で名前解決できることを確認する。

## 3. 結果と考察
設定後、以下のコマンドを実行し、サービス管理、ログ確認、NTP同期、システム更新、sudo設定、Guest Agent、ネットワーク設定が正しく反映されていることを確認する。

```bash
sudo apt update
journalctl -u systemd-timesyncd -n 30
dmesg | tail -n 30
timedatectl status
systemctl status systemd-timesyncd
sudo -l
ip a
systemctl status qemu-guest-agent
sudo netplan apply
ping -c 1 www.t3.metro-cit.internal
```

📷 **【エビデンス取得】** 以下の8点をスクリーンショットで取得し、図としてレポートに挿入・説明すること。

1. `apt update` または `apt upgrade` 実行後、更新が反映されたことがわかる画面
2. `systemctl list-units --type=service --state=running` を実行し、サービス一覧が確認できる画面
3. `journalctl -u systemd-timesyncd -n 30` などを実行し、サービスログが確認できる画面
4. `dmesg | tail -n 30` を実行し、起動ログ（カーネルメッセージ）が確認できる画面
5. `timedatectl status` または `systemctl status systemd-timesyncd` を実行し、NTP同期が有効であることが確認できる画面
6. `sudo -l` を実行し、sudo権限が確認できる画面
7. `ip a` コマンドを実行し、IPアドレスが表示されている画面全体
8. Windows 11 クライアントで `ping www.t3.metro-cit.internal` を実行し、hosts設定による名前解決が確認できる画面

## 4. 課題
1. GUI（デスクトップ環境）を持たない「Server版」を利用するメリットは何か。リソース（CPUやメモリ）の観点から考察せよ。
2. OS（基本ソフトウェア）は大きくクライアント用とサーバー用に分けられるが、サーバー用OSとしてLinux系（Unix系）が多く用いられる理由を歴史的背景やネットワーク機能との親和性から説明せよ。
3. 本演習で確認したIPアドレス（IPv4）は32ビットのアドレス体系である。IPv4アドレスの枯渇問題への対策として用いられている「グローバルアドレス」と「プライベートアドレス」の違いと、組織内（LAN）通信の仕組みについて説明せよ。
4. 情報機器をネットワークに参加させるために必要なパラメータのうち「デフォルトゲートウェイ」はどのような役割を持つか、内部（LAN内）通信と外部通信の違いに触れて説明せよ。
5. ネットワーク設定の変更前に設定ファイルのバックアップを作成した。システム設定ファイルを編集する前にバックアップを取ることがなぜ重要なのか、考察せよ。
6. OSやソフトウェアのアップデートを実施することには、どのような利点があるか。セキュリティと保守運用の観点から説明せよ。
7. QEMU Guest AgentをProxmoxとゲストOSの両方で有効化することで、どのような管理上の利点が得られるか考察せよ。
8. Linuxでサービスを管理する際、`systemctl status`、`restart`、`is-enabled` を使い分ける意義を説明せよ。
9. `sudo` を用いた権限管理が、常時rootで作業する方法と比べて安全である理由を説明せよ。
10. サーバー時刻をNTPで同期しておくことが、ログ解析やセキュリティ運用において重要な理由を説明せよ。
11. `journalctl` を用いてログを確認することが、障害調査や保守運用で有効である理由を説明せよ。
12. hostsファイルによる名前解決を利用する利点と、運用上の注意点を説明せよ。

