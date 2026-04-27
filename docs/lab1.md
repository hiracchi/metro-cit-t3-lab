# レポート課題1（1日目）：仮想化基盤への接続、OSインストール、初期設定

本実験は1回分（4時限分 × 1）を想定している。

実験レポートのファイル名は T3情報工学1_[学生番号]_[氏名].pdf とすること。


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
Proxmoxの基本操作を学び、Linuxサーバー（Ubuntu Server）を最小構成で立ち上げたうえで、VMスナップショットの取得と確認、サービス管理の基本操作、サーバーログ確認、systemd-timesyncdによるNTPクライアント設定、システムの最新化、sudo設定の確認、QEMU Guest Agentの導入、ネットワーク設定、hostsファイル設定を行う。

## 作業区分ラベル
- `[Webブラウザ(Proxmox)]`: Proxmoxの画面操作
- `[Windows 11 ターミナル]`: Windows 11 の PowerShell 操作
- `[Ubuntuサーバー ターミナル]`: Ubuntu Server 上のコマンド操作

## 2. 実験方法（使用機器と手順）
1. [Webブラウザ(Proxmox)] Proxmoxへのログイン

    Windows 11クライアントのWebブラウザ（Chrome）から指定されたURLへアクセスし、以下の設定でログインする。

    - User name: 指定されたユーザー名
    - Password: 指定されたパスワード
    - Realm: Proxmox VE authentication server を選択
    - Language: 日本語を選択

2. [Webブラウザ(Proxmox)] VM（仮想マシン）の作成

    右上の「VMを作成」をクリックし、以下のように設定する。

    - 全般
        - ノード: 可能なノード
        - VM ID: 100 + [学生番号下2桁]
        - 名前: t3lab-[学生番号下2桁]
        - HAに追加: オフ
        - リソースプール: 空欄
    - OS
        - CD/DVDイメージファイル(iso)を使用
          - ストレージ: gennai-nfs-01
          - ISOイメージ: ubuntu-24.04.4-live-server-amd64.iso
        - ゲストOS:
          - 種別: Linux
          - バージョン: 6.x - 2.6 Kernel
    - システム
        - デフォルトでOK
    - ディスク
        - デフォルトでOK
    - CPU
        - ソケット: 1
        - コア: 2
        - 種別: host (最下段)
    - メモリ
        - 1024
    - ネットワーク
        - ブリッジ: vmbr0
        - その他: デフォルト

    以上を設定の上、確認タブで完了を押すとVM作成が開始する。
    しばらく待つと、サーバーツリーの下にVMが作成される。

3. [Webブラウザ(Proxmox)] OSインストール

    1. 作成したVMを左のツリーから選択し、右上の[開始]をクリックするとVMが起動する
    2. VMメニューのコンソールを選択すると、VMのコンソールが表示される
    3. 上記でISOイメージを選択しているので、ISOからVMが起動する
    4. OSインストーラーからの質問に答えて、OSをインストールする
       - Language: English
       - Installer update: Continue without updating
       - Keyboard configuration:
         - Layout: Japanese
         - Variant: Japanese
       - the type of instlation: Ubuntu Server
       - Network configuration: そのまま(デフォルト)
       - Proxy configuration: 空欄
       - Ubuntu archive mirror configuration: そのまま(デフォルト)
       - Guided storage configuration:
         - Use an entire disk を選択
         - Set up this disk as an LVM group のチェックを外す
       - Storage configuration: そのまま(デフォルト)
       - Confirm destructive action: 問題なければ [Continue]
       - Profile configuration:
         - Your name: t3admin
         - Your servers name: t3lab-[学生番号下2桁]
         - Pick a username: t3admin
         - Choose a password: [指定されたパスワード]
         - Confirm your password: 上記と同一のもの
       - Upgrade to Ubuntu Pro:
         - Skip for now を選択
       - SSH configuration:
         - Install OpenSSH server を選択
       - Featured server snaps:
         - 何も選択せずに [Done]
    5. Installing system でインストールのログが流れる。
    6. しばらくすると [Reboot Now] が表示されるので、選択する。`press ENTER:` が表示されるので指示に従う。
    7. 再起動するとインストールしたUbuntuからブートする。ブートログが表示される。
    8. しばらくすると`login:`が表示されるので、OSインストール時に設定したユーザーとパスワードでログインする。
    9. `sudo shutdown -h now` を入力し、パスワードを入力後、シャットダウンする。
    10. ハードウェアから不要なISOイメージを削除する。

4. [Webブラウザ(Proxmox)] VMスナップショットの取得と確認

    初期状態へ戻せるように、VMのスナップショットを取得する。VMを選択し、`Snapshots` タブからスナップショット名（例: `day1-initial`）と説明を入力して作成する。

    作成後、スナップショット一覧に `day1-initial` が表示されること、対象VMに紐づいていることを確認する。

    今後、大きな作業を完了するたびにスナップショットを取るようにする。

5. [Webブラウザ(Proxmox)] VMの起動と停止、操作

    以降、VMを立ち上げるときは[開始]、停止するときはOS上からシャットダウンするか[シャットダウン]をクリックする。
    VMの操作はコンソールで行える。

6. [Ubuntuサーバー ターミナル] サービスの一覧確認と管理方法の確認

    サーバーで動作しているサービスの一覧と、管理コマンドを確認する。systemd-timesyncdサービスを例に、状態確認と再起動を試す。

    ```bash
    systemctl list-units --type=service --state=running
    systemctl status systemd-timesyncd
    sudo systemctl restart systemd-timesyncd
    systemctl is-enabled systemd-timesyncd
    ```

7. [Ubuntuサーバー ターミナル] サーバーログの確認

    障害調査の基本として、`journalctl` と `dmesg` によるログ確認方法を習得する。サービス全体の直近ログ、特定サービス（systemd-timesyncd）のログ、起動ログ（カーネルメッセージ）を確認する。

    ```bash
    journalctl -n 30
    journalctl -u systemd-timesyncd -n 30
    journalctl -p warning -n 30
    dmesg | tail -n 30
    ```

8. [Ubuntuサーバー ターミナル] systemd-timesyncd（NTPクライアント）の設定

    サーバー時刻を自動同期するため、設定ファイルを編集してNTPサーバーを `192.168.100.1` に指定する。

    ```bash
    sudo nano /etc/systemd/timesyncd.conf
    ```

    `[Time]` セクションの `NTP=` 行のコメントアウトを外し、以下のように設定する。

    ```ini
    [Time]
    NTP=192.168.100.1
    ```

    設定保存後、NTP同期を有効化し、サービスを再起動して同期先を確認する。

    ```bash
    sudo timedatectl set-ntp true
    sudo systemctl restart systemd-timesyncd
    systemctl status systemd-timesyncd
    timedatectl show-timesync --all
    ```

9. [Ubuntuサーバー ターミナル] システムの最新化

    OSインストール完了後にログインし、パッケージ一覧とインストール済みソフトウェアを更新する。

    ```bash
    sudo apt update
    sudo apt upgrade -y
    sudo apt autoremove -y
    ```

10. [Ubuntuサーバー ターミナル] sudoの設定確認

    管理作業に必要な `sudo` が使えることを確認し、必要に応じて sudoers 設定を確認する。

    ```bash
    sudo -l
    id
    sudo visudo
    ```

    `sudo visudo` では `%sudo ALL=(ALL:ALL) ALL` のような設定を確認する。

11. [Webブラウザ(Proxmox)] QEMU Guest Agentの有効化

    ProxmoxのVM設定画面で、対象仮想マシンの `Options` から `QEMU Guest Agent` を有効化する。

12. [Ubuntuサーバー ターミナル] QEMU Guest Agentのインストール

    Ubuntu側で以下のコマンドを実行してGuest Agentをインストールし、起動状態を確認する。

    ```bash
    sudo apt install -y qemu-guest-agent
    sudo systemctl enable --now qemu-guest-agent
    systemctl status qemu-guest-agent
    ```

13. [Ubuntuサーバー ターミナル] ネットワーク状態の確認

    OSインストール完了後、コンソールでログインし、割り当てられたIPアドレスを確認する。

    ```bash
    ip a
    ```

14. [Ubuntuサーバー ターミナル] 固定IP設定

    1. 現在のNetplan設定ファイルをコピーする。

        ```bash
        cd /etc/netplan
        sudo cp 00-installer-config.yaml 99-custom.yaml
        ```

    2. `vi` や `nano` エディタで設定ファイル`99-custom.yaml`を編集し、指定されたIPアドレス、ゲートウェイ、DNSを設定する。IPアドレスのxxxは100+学生番号の下2桁とする。


        ```text
        network:
          renderer: networkd
          ethernets:
            ens18:  # NIC device name. 
              addresses:
                - 192.168.100.xxx/24 # IP Address
              dhcp4: false           # Disable DHCP
              routes:                # Gateway configuration
                - to: default        # Any address should go to this gateway. 
                  via: 192.168.100.1 # Gateway server address
              nameservers:
                addresses: [192.168.100.1] # List of nameservers. 
        ```


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
ping -c 3 www.google.co.jp
```

📷 **【エビデンス取得】** 以下の8点をスクリーンショットで取得し、図としてレポートに挿入・説明すること。

1. Proxmox の `Snapshots` 画面で `day1-initial` などのスナップショットが作成・表示されている画面
2. `apt update` または `apt upgrade` 実行後、更新が反映されたことがわかる画面
3. `systemctl list-units --type=service --state=running` を実行し、サービス一覧が確認できる画面
4. `journalctl -u systemd-timesyncd -n 30` などを実行し、サービスログが確認できる画面
5. `dmesg | tail -n 30` を実行し、起動ログ（カーネルメッセージ）が確認できる画面
6. `timedatectl status` または `systemctl status systemd-timesyncd` を実行し、NTP同期が有効であることが確認できる画面
7. `sudo -l` を実行し、sudo権限が確認できる画面
8. `ip a` コマンドを実行し、IPアドレスが表示されている画面全体

## 4. 課題
1. OS（基本ソフトウェア）は大きくクライアント用とサーバー用に分けられるが、サーバー用OSとしてLinux系（Unix系）が多く用いられる理由を歴史的背景やネットワーク機能との親和性から説明せよ。
2. OSやソフトウェアのアップデートを実施することには、どのような利点があるか。セキュリティと保守運用の観点から説明せよ。
3. `sudo` を用いた権限管理が、常時rootで作業する方法と比べて安全である理由を説明せよ。
