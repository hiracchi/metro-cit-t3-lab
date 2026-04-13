# レポート課題2（2日目）：ファイアウォール設定・SSH鍵認証の確認

## 【基礎知識】ファイアウォールの役割
ファイアウォールは、サーバーへの通信のうち、必要なものだけを許可し、不要な通信を遮断するための仕組みである。Linuxでは `ufw` を用いて設定することが多く、SSH接続やWeb公開に必要なポートだけを開放することで、外部からの不正なアクセスや意図しない通信のリスクを低減できる。

## 【基礎知識】SSHと鍵認証
SSHはサーバーへ遠隔接続して管理するための仕組みであり、パスワード認証よりも公開鍵認証のほうが安全である。公開鍵をサーバーに登録し、`sshd_config` でパスワード認証を無効化することで、総当たり攻撃などのリスクを低減できる。

## 1. 目的
ファイアウォールの基本設定とSSHの鍵認証設定を行い、安全にサーバーへ接続できるリモート管理環境を構築する。

## 作業区分ラベル
- `[Webブラウザ(Proxmox)]`: Proxmoxの画面操作
- `[Windows 11 ターミナル]`: Windows 11 の PowerShell 操作
- `[Ubuntuサーバー ターミナル]`: Ubuntu Server 上のコマンド操作

## 2. 実験方法（使用機器と手順）

1. [Ubuntuサーバー ターミナル] ファイアウォールの設定

    `ufw` を用いて必要なポートのみを開放する。SSH（22/tcp）と後続の実験で利用するHTTP（80/tcp）を許可し、それ以外の不要な通信を遮断する。

    ```bash
    sudo ufw allow 22/tcp
    sudo ufw enable
    ```

1. [Ubuntuサーバー ターミナル] ファイアウォール設定の確認

    設定後、許可したポートのみが開放されていることを確認する。

    ```bash
    sudo ufw status numbered
    sudo ss -tuln
    ```

1. [Windows 11 ターミナル] クライアントからの通信確認

    Windows 11 のクライアント端末からサーバーに対して、許可したポートには接続でき、許可していないポートには接続できないことを確認する。PowerShell を用いて、外部から見た通信可否を確認する。

    ```powershell
    Test-NetConnection 192.168.1.10 -Port 22
    ```

    この結果から、22/tcp と 80/tcp には接続できること、未許可の 443/tcp には接続できないことを確認する。

1. [Windows 11 ターミナル] Windows 11でのSSH鍵作成

    Windows 11 のPowerShellでSSH鍵ペアを作成する。鍵の保存場所とファイル名を指定し、作成後に公開鍵ファイルが生成されていることを確認する。

    ```powershell
    ssh-keygen -t ed25519 -C "t3-lab" -f $env:USERPROFILE\.ssh\id_ed25519_t3
    Get-ChildItem $env:USERPROFILE\.ssh\id_ed25519_t3*
    ```

    必要に応じて `id_ed25519_t3.pub` の内容を確認し、次の手順でサーバーへ登録する。

1. [Ubuntuサーバー ターミナル] SSH鍵認証の設定

    作成した公開鍵（`id_ed25519_t3.pub`）をサーバーへ登録する。その後、サーバー側の `/etc/ssh/sshd_config` を編集し、鍵認証を有効化したうえでパスワード認証を無効化する。

    例（サーバー側設定）:

    ```bash
    sudo sshd -t
    sudo systemctl restart ssh
    sudo systemctl status ssh
    ```

    設定後、`[Windows 11 ターミナル]` からSSH接続し、パスワードなしでログインできることを確認する。

1. [Windows 11 ターミナル] SSHファイル転送（scp）演習

    Windows 11 クライアントからサーバーへファイルを送信し、サーバーからクライアントへファイルを受信する。送受信の両方を実施し、リモート運用時の基本操作を確認する。

    ```powershell
    scp .\sample.txt student@192.168.1.10:~/sample_from_win.txt
    scp student@192.168.1.10:~/sample_from_win.txt .\sample_from_server.txt
    ```

    転送後、Windows側とサーバー側でファイルの存在を確認する。

## 3. 結果と考察
ファイアウォール設定とSSH鍵認証設定後、以下のコマンドを実行し、安全な通信制御とリモート接続が正しく行えることを確認する。

**[Ubuntuサーバー ターミナル]**

```bash
sudo ufw status numbered
sudo sshd -t
sudo systemctl status ssh
ls -l ~/sample_from_win.txt
```

**[Windows 11 ターミナル]**

```powershell
ssh-keygen -t ed25519 -C "t3-lab" -f $env:USERPROFILE\.ssh\id_ed25519_t3
Test-NetConnection 192.168.1.10 -Port 22
scp .\sample.txt student@192.168.1.10:~/sample_from_win.txt
scp student@192.168.1.10:~/sample_from_win.txt .\sample_from_server.txt
```

📷 **【エビデンス取得】** 以下の6点をスクリーンショットで取得し、図としてレポートに挿入・説明すること。

1. `sudo ufw status numbered` を実行し、必要なポートのみが許可されていることが確認できる画面
2. Windows 11 クライアントで `ssh-keygen -t ed25519 -C "t3-lab" -f $env:USERPROFILE\.ssh\id_ed25519_t3` と `Get-ChildItem $env:USERPROFILE\.ssh\id_ed25519_t3*` を実行し、鍵ペアが生成されたことが確認できる画面
3. `sudo sshd -t` と `sudo systemctl status ssh` を実行し、SSH設定が有効でサービスが動作していることが確認できる画面
4. Windows 11 クライアントからSSH接続し、パスワードなしでログインできることが確認できる画面
5. Windows 11 クライアントで `scp .\sample.txt student@192.168.1.10:~/sample_from_win.txt` と `scp student@192.168.1.10:~/sample_from_win.txt .\sample_from_server.txt` を実行し、送受信が成功したことが確認できる画面
6. Windows 11 クライアントで `Test-NetConnection 192.168.1.10 -Port 22`、`Test-NetConnection 192.168.1.10 -Port 80`、`Test-NetConnection 192.168.1.10 -Port 443` を実行し、許可した通信と遮断された通信の違いが確認できる画面

## 4. 課題
1. ファイアウォール設定にて、ポート22（SSH）を許可した理由を説明せよ。また、不要なポートを開放したままにすることの危険性を考察せよ。
2. パスワード認証を禁止し、鍵認証のみに制限するセキュリティ上の理由は何か。

