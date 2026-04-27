# 付録：Linuxコマンド・パス表記・エディタ操作の基礎

この付録では、実験で頻繁に使う Linux（bash系シェル）の基本操作をまとめる。あわせて、Windows 11 や macOS との違いも示す。

## 1. Linux（bash）基本コマンド

### 1-1. まず覚える移動・確認コマンド

```bash
pwd                 # 現在の作業ディレクトリを表示
ls                  # ファイル・ディレクトリ一覧を表示
ls -la              # 詳細表示（隠しファイル含む）
cd /path/to/dir     # ディレクトリ移動
cd ~                # ホームディレクトリへ移動
cd ..               # 1つ上の階層へ移動
```

### 1-2. ファイル操作コマンド

```bash
mkdir testdir               # ディレクトリ作成
touch sample.txt            # 空ファイル作成
cp a.txt b.txt              # ファイルコピー
cp -r dir1 dir2             # ディレクトリコピー
mv old.txt new.txt          # 名前変更/移動
rm file.txt                 # ファイル削除
rm -r dir                   # ディレクトリ削除
cat file.txt                # ファイル内容表示
less file.txt               # ページ送り表示（qで終了）
```

### 1-3. 検索・フィルタ・ログ確認

```bash
grep "error" app.log               # 文字列検索
grep -R "Bind9" /etc               # 再帰検索
find /etc -name "*.conf"           # ファイル検索
tail -n 50 /var/log/syslog          # 末尾50行表示
journalctl -u ssh -n 30             # サービスログ表示
```

### 1-4. パイプとリダイレクト

```bash
cat access.log | grep 192.168.100.xxx  # パイプ（|）で結果を渡す
echo "hello" > out.txt             # 上書き出力
echo "world" >> out.txt            # 追記出力
command 2> err.log                  # エラー出力のみ保存
```

## 2. ファイル・ディレクトリの表し方（パス）

### 2-1. 絶対パスと相対パス

- 絶対パス: ルート `/` から始まるパス
例: `/etc/ssh/sshd_config`
- 相対パス: 現在位置を基準とするパス
例: `./docs/lab1.md`, `../lab2.md`

### 2-2. よく使う記号

- `.`: 現在のディレクトリ
- `..`: 1つ上のディレクトリ
- `~`: 現在ユーザーのホームディレクトリ
- `/`: 区切り文字（Linux/macOS）

### 2-3. Windows / macOS / Linux のパス差分

- Linux: `/home/student/.ssh/id_ed25519`
- macOS: `/Users/user/.ssh/id_ed25519`
- Windows: `C:\Users\user\.ssh\id_ed25519`（PowerShellでは `/` も扱える場合がある）

### 2-4. ファイル・ディレクトリの属性

Linuxでは、ファイルやディレクトリに次の属性がある。

- 種別: 通常ファイル、ディレクトリ、シンボリックリンクなど
- 権限: 読み取り（r）、書き込み（w）、実行（x）
- 所有者: owner（ユーザー）
- 所有グループ: group
- 更新時刻: 最終更新日時

確認コマンド例:

```bash
ls -l           # 種別、権限、所有者、サイズ、時刻を表示
stat file.txt   # 属性を詳細表示
```

`ls -l` の先頭例 `-rw-r--r--` の見方:

- 1文字目: 種別（`-` は通常ファイル、`d` はディレクトリ）
- 2〜4文字目: 所有者権限
- 5〜7文字目: グループ権限
- 8〜10文字目: その他ユーザー権限

### 2-5. 属性に関する基本操作

```bash
chmod 644 file.txt                 # 権限変更（rw-r--r--）
chmod 755 script.sh                # 実行権限付与（rwxr-xr-x）
chown student file.txt             # 所有者変更
chown student:student file.txt     # 所有者とグループ変更
chgrp www-data file.txt            # グループ変更
ln -s /path/src /path/link         # シンボリックリンク作成
```

再帰的に変更する場合は `-R` を使う（例: `chmod -R 755 dir`）。ただし、対象を誤ると広範囲に影響するため注意する。

## 3. nano の基本操作

`nano` は初心者向けのシンプルなターミナルエディタ。

```bash
nano /etc/hosts
```

- 文字入力: そのまま入力
- 保存: `Ctrl + O` → Enter
- 終了: `Ctrl + X`
- 検索: `Ctrl + W`
- 切り取り（行）: `Ctrl + K`
- 貼り付け: `Ctrl + U`

## 4. vim の基本操作

`vim` は高機能エディタ。モードの違いを理解することが重要。

```bash
vim /etc/netplan/00-installer-config.yaml
```

### 4-1. 主要モード

- ノーマルモード: 起動直後のモード（移動・削除・保存命令）
- 挿入モード: 文字入力するモード

### 4-2. 最低限の操作

- 挿入モードへ: `i`
- ノーマルモードへ戻る: `Esc`
- 保存して終了: `:wq` → Enter
- 保存せず終了: `:q!` → Enter
- 検索: `/文字列` → Enter
- 1行削除: `dd`

## 5. Windows 11 / macOS との差異（実習で重要な点）

### 5-1. シェルとコマンド

- Linux: bash（または sh 互換シェル）
- macOS: zsh が標準（bash系文法は多くが共通）
- Windows: PowerShell が主流（コマンド名やオプション体系が異なる）

例:

- Linux/macOS: `ls`, `cp`, `mv`, `cat`
- PowerShell: `Get-ChildItem`, `Copy-Item`, `Move-Item`, `Get-Content`

### 5-2. 権限昇格

- Linux/macOS: `sudo` を利用
- Windows: 管理者権限の PowerShell / コマンドプロンプトを起動

### 5-3. 改行コード

- Linux/macOS: LF
- Windows: CRLF

シェルスクリプトをWindowsで編集した場合、改行コードの違いで実行エラーになることがある。

### 5-4. SSH鍵の保存場所

- Linux/macOS: `~/.ssh/`
- Windows: `$env:USERPROFILE\.ssh\`

### 5-5. 属性・権限の扱いの差分

- Linux: `chmod`、`chown` で権限/所有者を直接管理する。
- macOS: Linuxに近いが、拡張属性（`xattr`）やACLが使われる場合がある。
- Windows: NTFSのACLによる権限管理が中心で、Linuxの `rwx` とは表現が異なる。

実習でLinuxサーバー設定ファイルを扱うときは、Linux側で `ls -l` と `stat` を確認してから編集するのが安全である。

## 6. 実習向けの運用上の注意

- 破壊的コマンド（`rm -rf` など）は対象パスを `pwd` と `ls` で確認してから実行する。
- 設定変更前はバックアップを作成する。
- `sudo` 実行時は、コマンドと対象ファイルを再確認する。
- エラー時は、まず `systemctl status` と `journalctl` でログを確認する。
