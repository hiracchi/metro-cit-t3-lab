# 付録2：DNSサーバー（BIND9）構築の補足

この付録では、BIND9を用いたDNSサーバー構築の基本手順と確認方法をまとめる。4日目のWebサーバー演習とあわせて参照すること。

## 【基礎知識】DNSと名前解決
DNSはホスト名とIPアドレスを対応付ける仕組みである。サーバーを名前で利用できるようにすることで、IPアドレス変更時の運用負荷を下げられる。hostsファイルとの違いも含めて確認する。

## 1. 目的
BIND9を用いたDNSサーバーの基本設定を行い、ホスト名とIPアドレスの対応付け、および名前解決の確認方法を習得する。

## 作業区分ラベル
- `[Webブラウザ(Proxmox)]`: Proxmoxの画面操作
- `[Windows 11 ターミナル]`: Windows 11 の PowerShell 操作
- `[Ubuntuサーバー ターミナル]`: Ubuntu Server 上のコマンド操作

## 2. 実験方法（使用機器と手順）
1. [Ubuntuサーバー ターミナル] DNSサーバー（BIND9）のインストール

```bash
sudo apt install -y bind9 dnsutils
sudo ufw allow 53/udp
```

1. [Ubuntuサーバー ターミナル] DNSゾーン設定

`/etc/bind/named.conf.local` に `t3.metro-cit.internal` 用のゾーン設定を追加し、ゾーンファイルを作成して `www.t3.metro-cit.internal` を自分のサーバーIPに対応付ける。

1. [Ubuntuサーバー ターミナル] DNS設定確認と再起動

```bash
sudo named-checkconf
sudo named-checkzone t3.metro-cit.internal /etc/bind/db.t3.metro-cit.internal
sudo systemctl restart bind9
sudo systemctl status bind9
```

1. [Windows 11 ターミナル] 名前解決確認

1日目で設定した hosts を前提に、DNSサーバーへの問い合わせ結果を確認する。

```powershell
nslookup www.t3.metro-cit.internal 192.168.100.xxx
```

## 3. 結果と考察
**[Ubuntuサーバー ターミナル] 確認コマンド:**

```bash
dig @localhost www.t3.metro-cit.internal
systemctl status bind9
```

**[Windows 11 ターミナル] 確認コマンド:**

```powershell
nslookup www.t3.metro-cit.internal 192.168.100.xxx
```

📷 **【エビデンス取得】** 以下の3点をキャプチャし、図表のルールにしたがってレポートにまとめること。

1. `dig @localhost www.t3.metro-cit.internal` を実行し、DNS名前解決結果が表示される画面
2. `systemctl status bind9` を実行し、DNSサービスが動作していることがわかる画面
3. `nslookup www.t3.metro-cit.internal 192.168.100.xxx` をWindows 11 から実行し、名前解決できる画面

## 4. 課題
1. DNSの「正引き」と「逆引き」の違いは何か。
2. URL「`http://www.metro-cit.ac.jp`」にアクセスする際、DNSはどのような順序でIPアドレスを問い合わせるか。ドメインの階層構造（.jp、.ac、.metro-cit等）を含めて説明せよ。
3. DNS設定ファイルの編集後に `named-checkconf` や `named-checkzone` を実行することには、どのような意義があるか考察せよ。
4. hostsファイルによる名前解決とDNSによる名前解決には、それぞれどのような利点と制約があるか。
