# Chapter 2　コマンドライン
- レベル:初級
- 見込み時間:30分
- コンテンツ概要
    - コンテナの操作
    - コンテナイメージの操作
    - イメージの構築

---

## 事前作業
``` title="一般ユーザー(ユーザー名 rhel)にスイッチする"
# su - rhel
```

(コピペ用)
``` { .yaml .copy } 
su - rhel
```

/// details | 出力結果
    type: success
```
# su - rhel
$
```
///

---

## 2.1 コンテナの操作
### コンテナの探索
(このコンテンツは本書 2.1.1に該当します)

`podman run`コマンドを実行し、`ubi8/httpd-24` イメージをプルして実行します
```
$ podman run -ti --rm registry.access.redhat.com/ubi8/httpd-24 bash
```

(コピペ用)
``` { .yaml .copy } 
podman run -ti --rm registry.access.redhat.com/ubi8/httpd-24 bash
```

/// details | 出力結果
    type: success
```
$ podman run -ti --rm registry.access.redhat.com/ubi8/httpd-24 bash
Trying to pull registry.access.redhat.com/ubi8/httpd-24:latest...
Getting image source signatures
Checking if image destination supports signatures
Copying blob 9ece777c9660 done  
Copying blob 70de3d8fc2c6 done  
Copying blob b653248f5bcb done  
Copying config c4127096ce done  
Writing manifest to image destination
Storing signatures
bash-4.4$ 
```
///

コンテナ内でOSの情報を参照します
```
bash-4.4$ grep PRETTY_NAME /etc/os-release
```

(コピペ用)
``` { .yaml .copy } 
grep PRETTY_NAME /etc/os-release
```

/// details | 出力結果
    type: success
```
bash-4.4$ grep PRETTY_NAME /etc/os-release
PRETTY_NAME="Red Hat Enterprise Linux 8.8 (Ootpa)"
```
///

コンテナ内の`/usr/bin` のカウントを行います
```
bash-4.4$ ls /usr/bin/ | wc -l
```

(コピペ用)
``` { .yaml .copy } 
ls /usr/bin/ | wc -l
```

/// details | 出力結果
    type: success
```
bash-4.4$ ls /usr/bin/ | wc -l
526
```
///

ホストOS上で同じように実行してみます。`exit`でコンテナからホストOSに戻り、コマンドを実行します。
```
bash-4.4$ exit
$ grep PRETTY_NAME /etc/os-release
$ ls /usr/bin/ | wc -l
```

(コピペ用)
``` { .yaml .copy } 
exit
```

``` { .yaml .copy } 
grep PRETTY_NAME /etc/os-release
```

``` { .yaml .copy } 
ls /usr/bin/ | wc -l
```

//// details | 出力結果
    type: success
/// admonition
    type: info
ラボ環境以外ではホストOSの状態により出力される数字は異なります
///
```
bash-4.4$ exit
exit
$ grep PRETTY_NAME /etc/os-release
PRETTY_NAME="Red Hat Enterprise Linux 9.2 (Plow)"
$ ls /usr/bin/ | wc -l
1022
```
////

### コンテナ化したアプリケーションの実行
(このコンテンツは本書 2.1.2に該当します)

`デタッチモード`、ポート `8080`、名前 `myapp`でコンテナを起動
```
$ podman run -d -p 8080:8080 --name myapp registry.access.redhat.com/ubi8/httpd-24
```

(コピペ用)
``` { .yaml .copy }
podman run -d -p 8080:8080 --name myapp registry.access.redhat.com/ubi8/httpd-24
```

//// details | 出力結果
    type: success
/// admonition
    type: info
出力される英数字はコンテナIDで、コンテナ毎に一意のものが出力されます
///
```
$ podman run -d -p 8080:8080 --name myapp registry.access.redhat.com/ubi8/httpd-24
e5558cfdda7d1c2c49c4d5ae59bbfcece64cb219b848ba78ae6cbba787ee8d07
```
////


`myapp`コンテナのポートの使用状況を確認します
```
$ podman port myapp
```

(コピペ用)
``` { .yaml .copy }
podman port myapp
```

/// details | 出力結果
    type: success
```
$ podman port myapp
8080/tcp -> 0.0.0.0:8080
```
///


`デタッチモード`、ポート `8081`、名前 `myapp1`でコンテナを起動
```
$ podman run -d -p 8081:8080 --name myapp1 registry.access.redhat.com/ubi8/httpd-24
```

(コピペ用)
``` { .yaml .copy }
podman run -d -p 8081:8080 --name myapp1 registry.access.redhat.com/ubi8/httpd-24
```

/// details | 出力結果
    type: success
```
$ podman run -d -p 8081:8080 --name myapp1 registry.access.redhat.com/ubi8/httpd-24
f6218c8fd423b6ed913fee1d8d612972e3fc4d0cf737a44e058f0fff0f68aeeb
```
///

/// admonition | （オプション問題）
    type: question
コンテナのポートの使用状況を確認する
///

起動中の全てのコンテナのポートの使用状況を確認してみましょう
```
$ podman port --all 
```

(コピペ用)
``` { .yaml .copy } 
podman port --all 
```

/// details | 出力結果
    type: success
```
$ podman port --all 
e5558cfdda7d    8080/tcp -> 0.0.0.0:8080
f6218c8fd423    8080/tcp -> 0.0.0.0:8081
```

`podman port`の出力内容は、`コンテナID`、`コンテナ自身のポート番号`/`プロトコル`、`ホストのIPアドレス`:`ホスト上のポート番号`、の順に表示されます。
///


### コンテナの停止
(このコンテンツは本書 2.1.3に該当します)

`myapp`コンテナを停止します
```
$ podman stop myapp
```

(コピペ用)
``` { .yaml .copy } 
podman stop myapp
```

/// details | 出力結果
    type: success
```
$ podman stop myapp
myapp
```
///


`myapp1`コンテナを停止します
```
$ podman stop -t 0 myapp1
```

(コピペ用)
``` { .yaml .copy } 
podman stop -t 0 myapp1
```

/// details | 出力結果
    type: success
```
$ podman stop -t 0 myapp1
myapp1
```
///


### コンテナの起動
(このコンテンツは本書 2.1.4に該当します)

停止したコンテナを再度起動します。`myapp`コンテナを起動します。
```
$ podman start myapp
```

(コピペ用)
``` { .yaml .copy }
podman start myapp
```

/// details | 出力結果
    type: success
```
$ podman start myapp
myapp
```
///

### コンテナのリスト表示
(このコンテンツは本書 2.1.5に該当します)

起動中のコンテナを確認します
```
$ podman ps
```

(コピペ用)
``` { .yaml .copy }
podman ps
```

/// details | 出力結果
    type: success
```
$ podman ps
CONTAINER ID  IMAGE                                            COMMAND               CREATED        STATUS         PORTS                   NAMES
e5558cfdda7d  registry.access.redhat.com/ubi8/httpd-24:latest  /usr/bin/run-http...  9 minutes ago  Up 23 seconds  0.0.0.0:8080->8080/tcp  myapp
```
///


停止しているコンテナを含め、すべてのコンテナを表示します
```
$ podman ps --all
```

(コピペ用)
``` { .yaml .copy }
podman ps --all
```

/// details | 出力結果
    type: success
```
$ podman ps --all
CONTAINER ID  IMAGE                                            COMMAND               CREATED         STATUS                      PORTS                   NAMES
e5558cfdda7d  registry.access.redhat.com/ubi8/httpd-24:latest  /usr/bin/run-http...  10 minutes ago  Up About a minute           0.0.0.0:8080->8080/tcp  myapp
f6218c8fd423  registry.access.redhat.com/ubi8/httpd-24:latest  /usr/bin/run-http...  5 minutes ago   Exited (137) 2 minutes ago  0.0.0.0:8081->8080/tcp  myapp1
```
///

### コンテナの調査
(このコンテンツは本書 2.1.6に該当します)

`podman inspect`コマンドを使い、コンテナを調査します。
`myapp`コンテナについて調べてみます。
```
$ podman inspect myapp
```

(コピペ用)
``` { .yaml .copy }
podman inspect myapp
```

/// details | 出力結果
    type: success
```
$ podman inspect myapp
[
     {
          "Id": "e5558cfdda7d1c2c49c4d5ae59bbfcece64cb219b848ba78ae6cbba787ee8d07",
          "Created": "2023-09-03T19:58:13.302930658Z",
          "Path": "container-entrypoint",
          "Args": [
               "/usr/bin/run-httpd"
          ],
...
]
```
///


### コンテナの削除
(このコンテンツは本書 2.1.7に該当します)

`podman rm`コマンドを使用し、コンテナを削除します。
`myapp1`コンテナを削除します。
```
$ podman rm myapp1
```

(コピペ用)
``` { .yaml .copy }
podman rm myapp1
```

/// details | 出力結果
    type: success
```
$ podman rm myapp1
myapp1
```
///


### コンテナへの実行
(このコンテンツは本書 2.1.8に該当します)

`podman exec`コマンドを使用して、起動中のコンテナに新たにプロセスを実行できます。また、`--interactive (-i)`オプションを使用してコンテナ内でコマンドを実行できます。
`myapp`コンテナ内にhtmlファイルを作成します
```
$ podman exec -i myapp bash -c 'cat > /var/www/html/index.html' << _EOF
<html>
    <head>
    </head>
        <body>
            <h1>Hello World</h1>
        </body>
</html>
_EOF
```


(コピペ用)
``` { .yaml .copy }
podman exec -i myapp bash -c 'cat > /var/www/html/index.html' << _EOF
<html>
    <head>
    </head>
        <body>
            <h1>Hello World</h1>
        </body>
</html>
_EOF
```
/// admonition | 出力結果はありません

///

コンテナ内のファイルを確認
```
$ podman exec myapp cat /var/www/html/index.html
```

(コピペ用)
``` { .yaml .copy }
podman exec myapp cat /var/www/html/index.html
```

/// details | 出力結果
    type: success
```
$ podman exec myapp cat /var/www/html/index.html
<html>
    <head>
    </head>
        <body>
            <h1>Hello World</h1>
        </body>
</html>
```
///

### コンテナからイメージを作成
(このコンテンツは本書 2.1.9に該当します)

コンテナを停止します
```
$ podman stop myapp
```

(コピペ用)
``` { .yaml .copy }
podman stop myapp
```

/// details | 出力結果
    type: success
```
$ podman stop myapp
myapp
```
///

コンテナをコミットします。これにより、起動中の`myapp`コンテナから`myimage`コンテナイメージが作成されます。
```
$ podman commit myapp myimage
```

(コピペ用)
``` { .yaml .copy }
podman commit myapp myimage
```

/// details | 出力結果
    type: success
```
$ podman commit myapp myimage
Getting image source signatures
Copying blob 48bbc3bb7b39 skipped: already exists  
Copying blob ca07266b6575 skipped: already exists  
Copying blob 2860cc774137 skipped: already exists  
Copying blob 195affa04e51 done  
Copying config 7af708ab2c done  
Writing manifest to image destination
Storing signatures
7af708ab2cb1b896b2e6a946588a8607b1d02e1c4e7a0ac317fe6f205901e3c8
```
///

`myimage`コンテナイメージから`myapp1`コンテナを起動します
```
$ podman run -d --name myapp1 -p 8080:8080 myimage
```

(コピペ用)
``` { .yaml .copy }
podman run -d --name myapp1 -p 8080:8080 myimage
```

/// details | 出力結果
    type: success
```
podman run -d --name myapp1 -p 8080:8080 myimage
41862b662bdeb01bd6c2a2dac2e8b1d337a275d28fce720ad4a738650ee3112f
```
///

/// admonition | まとめ（2.1 コンテナの操作）
    type: abstract
このように...
///

## 2.2 コンテナイメージの操作
### コンテナとイメージの違い
(このコンテンツは本書 2.2.1に該当します)

`podman image tree`コマンドを実行します
```
$ podman image tree myimage
```

(コピペ用)
``` { .yaml .copy }
podman image tree myimage
```

/// details | 出力結果
    type: success
```
podman image tree myimage
Image ID: 7af708ab2cb1
Tags:     [localhost/myimage:latest]
Size:     453.8MB
Image Layers
├── ID: 48bbc3bb7b39 Size: 214.8MB
├── ID: c85189e46544 Size: 59.34MB
├── ID: 129ebf0074a0 Size: 179.5MB Top Layer of: [registry.access.redhat.com/ubi8/httpd-24:latest]
└── ID: eb3ca38cae75 Size:  51.2kB Top Layer of: [localhost/myimage:latest]
```
///

2つのコンテナイメージを比較します。
`myimage`コンテナイメージと元のコンテナイメージ(`ubi8/httpd-24`)との差分を表示します
```
$ podman image diff myimage ubi8/httpd-24
```

(コピペ用)
``` { .yaml .copy }
podman image diff myimage ubi8/httpd-24
```

//// details | 出力結果
    type: success
/// admonition | 実際にはコマンドの実行ごとに出力順が変化します
    type: info
///
```
$ podman image diff myimage ubi8/httpd-24
C /opt
C /opt/app-root
C /opt/app-root/etc
A /opt/app-root/etc/passwd
C /etc
C /etc/httpd
C /etc/httpd/conf.d
C /etc/httpd/conf.d/ssl.conf
C /etc/httpd/conf
C /etc/httpd/conf/httpd.conf
C /etc/httpd/tls
A /etc/httpd/tls/dhparams.pem
A /etc/httpd/tls/localhost.crt
A /etc/httpd/tls/localhost.key
C /etc/group
C /var
C /var/www
C /var/www/html
A /var/www/html/index.html
C /var/log
C /var/log/httpd
A /var/log/httpd/modsec_audit.log
A /var/log/httpd/modsec_debug.log
```
////


### イメージのリスト表示
(このコンテンツは本書 2.2.2に該当します)

イメージをリスト表示
```
$ podman images
```

(コピペ用)
``` { .yaml .copy }
podman images
```

/// details | 出力結果
    type: success
```
# podman images
REPOSITORY                                TAG         IMAGE ID      CREATED        SIZE
localhost/myimage                         latest      7af708ab2cb1  6 minutes ago  454 MB
registry.access.redhat.com/ubi8/httpd-24  latest      c4127096ce64  2 weeks ago    454 MB
```
///


### イメージの調査
(このコンテンツは本書 2.2.3に該当します)

`podman image inspect`コマンドを使いコンテナイメージを詳細に調べることができます。
`myimage`の調査を行います。
```
$ podman image inspect myimage
```

(コピペ用)
``` { .yaml .copy }
podman image inspect myimage
```

/// details | 出力結果
    type: success
```
$ podman image inspect myimage
[
     {
          "Id": "7af708ab2cb1b896b2e6a946588a8607b1d02e1c4e7a0ac317fe6f205901e3c8",
          "Digest": "sha256:2366a37f18bd462b09826a897f78fd3a523b57ba8d4330956dd0f2b0b7befe9a",
          "RepoTags": [
               "localhost/myimage:latest"
          ],
          "RepoDigests": [
               "localhost/myimage@sha256:2366a37f18bd462b09826a897f78fd3a523b57ba8d4330956dd0f2b0b7befe9a"
          ],
...
]
```
///


出力をフィルタリングし、特定の情報だけを出力できます。
`CMD`を出力します。
```
$ podman image inspect --format '{{ .Config.Cmd }}' myimage
```

(コピペ用)
``` { .yaml .copy }
podman image inspect --format '{{ .Config.Cmd }}' myimage
```

/// details | 出力結果
    type: success
```
$ podman image inspect --format '{{ .Config.Cmd }}' myimage
[/usr/bin/run-httpd]
```
///


`StopSignal`を出力します
```
$ podman image inspect --format '{{ .Config.StopSignal }}' myimage
```

(コピペ用)
``` { .yaml .copy }
podman image inspect --format '{{ .Config.StopSignal }}' myimage
```

/// details | 出力結果
    type: success
```
$ podman image inspect --format '{{ .Config.StopSignal }}' myimage

```
///

### イメージのプッシュ
(このコンテンツは本書 2.2.4に該当します)

コンテナトランスポート（単にトランスポートとも呼ぶ）
`podman run`コマンドを実行時、トランスポートを指定してコンテナを実行します。
```
$ podman run docker://registry.access.redhat.com/ubi8/httpd-24:latest echo hello
```

(コピペ用)
``` { .yaml .copy }
podman run docker://registry.access.redhat.com/ubi8/httpd-24:latest echo hello
```

/// details | 出力結果
    type: success
```
$ podman run docker://registry.access.redhat.com/ubi8/httpd-24:latest echo hello
hello
```
///

トランスポートを省略してコマンドを実行します。デフォルトは`docker://`
```
$ podman run registry.access.redhat.com/ubi8/httpd-24:latest echo hello
```

(コピペ用)
``` { .yaml .copy }
podman run registry.access.redhat.com/ubi8/httpd-24:latest echo hello
```

/// details | 出力結果
    type: success
```
$ podman run registry.access.redhat.com/ubi8/httpd-24:latest echo hello
hello
```
///

コンテナイメージのリストを出力します
```
$ podman images
```

(コピペ用)
``` { .yaml .copy }
podman images
```

/// details | 出力結果*
    type: success
```
$ podman images
REPOSITORY                                TAG         IMAGE ID      CREATED      SIZE
registry.access.redhat.com/ubi8/httpd-24  latest      c4127096ce64  2 weeks ago  454 MB
```
///


`myimage`コンテナイメージをコンテナレジストリにプッシュします
```
$ podman push myimage quay.io/rhatdan/myimage
```

出力結果*
```

```


### podman login：コンテナレジストリへのログイン
(このコンテンツは本書 2.2.5に該当します)

/// admonition | quay.ioとは
    type: info
quay.ioは、
以降のquay.ioへのログインやコンテナイメージのpushを行うためにはご自身のアカウントの作成が必要です。
Red Hatアカウントをお持ちの場合はそれを利用できます。
///

`quay.io`にログインします
```
$ podman login quay.io
```

(コピペ用)
``` { .yaml .copy }
podman login quay.io
```

/// details | 出力結果*
    type: success
```
$ podman login quay.io
Username: tnk4on
Password: 
Login Succeeded!
```
///


ローカルにキャッシュされた認証情報の確認します。

/// admonition | 注記
    type: tip
キャッシュされた情報は暗号化されていないのでファイルの取り扱いには注意しましょう。
///

```
$ cat /run/user/$UID/containers/auth.json
```

(コピペ用)
``` { .yaml .copy }
cat /run/user/$UID/containers/auth.json
```

/// details | 出力結果*
    type: success
```
$ cat /run/user/$UID/containers/auth.json
{
        "auths": {
                "quay.io": {
                        "auth": "xxxxxxxxxxxxxxxx"
                }
        }
}
```
///


`quay.io`からログアウトします
```
$ podman logout quay.io
```

(コピペ用)
``` { .yaml .copy }
$ podman logout quay.io
```

/// details | 出力結果*
    type: success
```
$ podman logout quay.io
Removed login credentials for quay.io
```
///

### イメージのタグ付け
(このコンテンツは本書 2.2.6に該当します)

コンテナイメージのリストを表示します
```
$ podman images
```

(コピペ用)
``` { .yaml .copy }
podman images
```

/// details | 出力結果
    type: success
```
$ podman images
REPOSITORY                                TAG         IMAGE ID      CREATED      SIZE
registry.access.redhat.com/ubi8/httpd-24  latest      c4127096ce64  2 weeks ago  454 MB
```
///

`podman tag`コマンドを使用してコンテナイメージに新しい名前（タグ）を追加できます。
レジストリ(`quay.io`)、ユーザー名(`rhatdan`)を付加した新しいタグを作成します。
```
$ podman tag myimage quay.io/rhatdan/myimage
$ podman images
```

(コピペ用)
``` { .yaml .copy }
podman tag myimage quay.io/rhatdan/myimage
```
``` { .yaml .copy }
podman images
```

/// details | 出力結果
    type: success
```
$ podman tag myimage quay.io/rhatdan/myimage
$ podman images
REPOSITORY                                TAG         IMAGE ID      CREATED         SIZE
localhost/myimage                         latest      c3c487bbbf7b  37 seconds ago  454 MB
quay.io/rhatdan/myimage                   latest      c3c487bbbf7b  37 seconds ago  454 MB
registry.access.redhat.com/ubi8/httpd-24  latest      c4127096ce64  2 weeks ago     454 MB
```
///


`quay.io`にユーザー名を指定してログインします*
```
$ podman login --username rhatdan quay.io
```

(コピペ用)
``` { .yaml .copy }
podman login --username rhatdan quay.io
```

/// details | 出力結果*
    type: success
```

```
///

`quay.io`にイメージをプッシュします
```
$ podman push quay.io/rhatdan/myimage
```

(コピペ用)
``` { .yaml .copy }
podman push quay.io/rhatdan/myimage
```
/// admonition | 出力結果はありません

///

`podman tag`でバージョンを指定したタグを作成します
```
$ podman tag quay.io/rhatdan/myimage quay.io/rhatdan/myimage:1.0
$ podman images
```

(コピペ用)
``` { .yaml .copy }
$ podman tag quay.io/rhatdan/myimage quay.io/rhatdan/myimage:1.0
```
``` { .yaml .copy }
podman images
```

/// details | 出力結果
    type: success
```
$ podman images
REPOSITORY                                TAG         IMAGE ID      CREATED        SIZE
localhost/myimage                         latest      c3c487bbbf7b  9 minutes ago  454 MB
quay.io/rhatdan/myimage                   latest      c3c487bbbf7b  9 minutes ago  454 MB
quay.io/rhatdan/myimage                   1.0         c3c487bbbf7b  9 minutes ago  454 MB
registry.access.redhat.com/ubi8/httpd-24  latest      c4127096ce64  2 weeks ago    454 MB
```
///

### イメージの削除
(このコンテンツは本書 2.2.7に該当します)

/// admonition | 以前に`podman commit myapp myimage`で作成したコンテナイメージは`localhost/myimage`という名前で保存されています。
    type: note
///

`localhost/myimage`タグを削除します
```
$ podman images
$ podman rmi localhost/myimage
$ podman images
```

(コピペ用)
``` { .yaml .copy }
podman images
```
``` { .yaml .copy }
podman rmi localhost/myimage
```
``` { .yaml .copy }
podman images
```


/// details | 出力結果
    type: success
```
$ podman images
REPOSITORY                                TAG         IMAGE ID      CREATED         SIZE
localhost/myimage                         latest      c3c487bbbf7b  10 minutes ago  454 MB
quay.io/rhatdan/myimage                   latest      c3c487bbbf7b  10 minutes ago  454 MB
quay.io/rhatdan/myimage                   1.0         c3c487bbbf7b  10 minutes ago  454 MB
registry.access.redhat.com/ubi8/httpd-24  latest      c4127096ce64  2 weeks ago     454 MB
$ podman rmi localhost/myimage
Untagged: localhost/myimage:latest
$ podman images
REPOSITORY                                TAG         IMAGE ID      CREATED         SIZE
quay.io/rhatdan/myimage                   latest      c3c487bbbf7b  10 minutes ago  454 MB
quay.io/rhatdan/myimage                   1.0         c3c487bbbf7b  10 minutes ago  454 MB
registry.access.redhat.com/ubi8/httpd-24  latest      c4127096ce64  2 weeks ago     454 MB
```
///


`myimage`および`myimage:1.0`を削除します

/// admonition | 訳注
1章で実行したコンテナが残っている場合は、myimageの削除前に停止する必要があります。
``` { .yaml .copy }
podman rm -f -t 0 myapp1
```
///

```
$ podman rmi myimage
$ podman rmi myimage:1.0
$ podman images
```

(コピペ用)
``` { .yaml .copy }
$ podman rmi myimage
```
``` { .yaml .copy }
$ podman rmi myimage:1.0
```
``` { .yaml .copy }
$ podman images
```

/// details | 出力結果
    type: success
```
$ podman rmi myimage
Untagged: quay.io/rhatdan/myimage:latest
$ podman rmi myimage:1.0
Untagged: quay.io/rhatdan/myimage:1.0
Deleted: c3c487bbbf7ba989432ae65facfac36ca34fbd75fde664b24cf6247bf844f955
$ podman images
REPOSITORY                                TAG         IMAGE ID      CREATED      SIZE
registry.access.redhat.com/ubi8/httpd-24  latest      c4127096ce64  2 weeks ago  454 MB
```
///


/// admonition | コンテナイメージを削除する別の方法
ID指定してコンテナイメージを削除することも可能です。
///


```
$ podman rmi edc479f58484
$ podman rmi edc479f58484 --force
```

`podman image prune -a`コマンドを使用すると、起動中のコンテナで使われているコンテナイメージ以外を全て削除できます。
ローカルにあるコンテナイメージを全て削除します。
```
$ podman image prune -a
$ podman images
```

(コピペ用)
``` { .yaml .copy }
podman image prune -a
```
``` { .yaml .copy }
podman images
```


参考
```
$ podman rmi edc479f58484
$ podman rmi edc479f58484 --force
```

全てのローカルイメージを削除する
```
$ podman image prune -a
$ podman images
```

出力結果
```
$ podman image prune -a
WARNING! This command removes all images without at least one container associated with them.
Are you sure you want to continue? [y/N] y
$ podman images
```


### イメージのプル
(このコンテンツは本書 2.2.8に該当します)

`quay.io`から`myimage`コンテナイメージをプルする
```
$ podman pull quay.io/rhatdan/myimage
```

(コピペ用)
``` { .yaml .copy }
podman pull quay.io/rhatdan/myimage
```

/// details | 出力結果
    type: success
```
$ podman pull quay.io/rhatdan/myimage
Trying to pull quay.io/rhatdan/myimage:latest...
Getting image source signatures
Copying blob e3460238f8a1 done  
Copying blob c7765172d3ce done  
Copying blob dfd8c625d022 done  
Copying blob 2b782a9ad894 done  
Copying blob a1eadb69adf1 done  
Copying config 2c7e43d880 done  
Writing manifest to image destination
Storing signatures
2c7e43d880382561ebae3fa06c7a1442d0da2912786d09ea9baaef87f73c29ae
```
///


Original
```
$ podman run -d -p 8080:8080 --name myapp registry.access.redhat.com/ubi8/httpd-24
```

(コピペ用)
``` { .yaml .copy }
podman run -d -p 8080:8080 --name myapp registry.access.redhat.com/ubi8/httpd-24
```

/// details | 出力結果
    type: success
```
$ podman run -d -p 8080:8080 --name myapp registry.access.redhat.com/ubi8/httpd-24
Trying to pull registry.access.redhat.com/ubi8/httpd-24:latest...
Getting image source signatures
Checking if image destination supports signatures
Copying blob 9ece777c9660 done  
Copying blob 70de3d8fc2c6 done  
Copying blob b653248f5bcb done  
Copying config c4127096ce done  
Writing manifest to image destination
Storing signatures
a68b01256feb009ba382174831f67a8f78af61024abe809c50d1ce6ae6f84131
```
///


コンテナイメージのリストを表示します
```
$ podman images
```

(コピペ用)
``` { .yaml .copy }
podman images
```

/// details | 出力結果
    type: success
```
$ podman images
REPOSITORY                                TAG         IMAGE ID      CREATED      SIZE
registry.access.redhat.com/ubi8/httpd-24  latest      c4127096ce64  2 weeks ago  454 MB
quay.io/rhatdan/myimage                   latest      2c7e43d88038  2 years ago  462 MB
```
///


Dockerはdocker.ioをハードコード
Podmanでは複数のレジストリが定義されている。パッケージングにより異なる。

Podmanの情報を出力
```
$ podman info
```

(コピペ用)
``` { .yaml .copy }
podman info
```

/// details | 出力結果
    type: success
```
$ podman info
...
registries:
  search:
  - registry.access.redhat.com
  - registry.redhat.io
  - docker.io
...
```
///

`podman create`コマンドを使うと、コンテナイメージからコンテナが作成されますがすぐには実行されません。
停止状態のコンテナを起動するには`podman start`コマンドを使用します。

短縮名でコンテナイメージを指定し、コンテナを作成します
```
$ podman create -p 8080:8080 ubi8/httpd-24
```

(コピペ用)
``` { .yaml .copy }
podman create -p 8080:8080 ubi8/httpd-24
```

/// admonition | 注記
    type: tip

RHELの場合*
```
$ podman create -p 8080:8080 ubi8/httpd-24
Resolved "ubi8/httpd-24" as an alias (/etc/containers/registries.conf.d/001-rhel-shortnames.conf)
Trying to pull registry.access.redhat.com/ubi8/httpd-24:latest...
Getting image source signatures
Checking if image destination supports signatures
Copying blob 9ece777c9660 done  
Copying blob 70de3d8fc2c6 done  
Copying blob b653248f5bcb done  
Copying config c4127096ce done  
Writing manifest to image destination
Storing signatures
94ad6a37b8fb40b13eb7038e5e174cc22bdd3fe8237a6fbd6a14bd87896528f8
```
- `"ubi8" = "registry.access.redhat.com/ubi8"` があるため解決してしまう


短縮名を設定しているファイルの内容を確認してみる
```
$ cat /etc/containers/registries.conf.d/000-shortnames.conf
```
出力結果
```
$ cat /etc/containers/registries.conf.d/000-shortnames.conf
[aliases]
...
  # centos
  "centos" = "quay.io/centos/centos"
  # containers
  "skopeo" = "quay.io/skopeo/stable"
  "buildah" = "quay.io/buildah/stable"
  "podman" = "quay.io/podman/stable"
...
```
///

### イメージの検索
(このコンテンツは本書 2.2.9に該当します)

リポジトリregistry.access.redhat.comで、名前にhttpdという文字列を含むイメージを検索します
```
$ podman search registry.access.redhat.com/httpd
```

(コピペ用)
``` { .yaml .copy }
podman search registry.access.redhat.com/httpd
```

/// details | 出力結果
    type: success
```
$ podman search registry.access.redhat.com/httpd
NAME                                                                         DESCRIPTION
registry.access.redhat.com/rhscl/httpd-24-rhel7                              Apache HTTP 2.4 Server
registry.access.redhat.com/ubi8/httpd-24                                     Platform for running Apache httpd 2.4 or bui...
registry.access.redhat.com/ubi9/httpd-24                                     rhcc_registry.access.redhat.com_ubi9/httpd-2...
registry.access.redhat.com/cloudforms46-beta/cfme-openshift-httpd            CloudForms is a management and automation pl...
registry.access.redhat.com/cloudforms46/cfme-openshift-httpd                 Web Server image for a multi-pod Red Hat® C...
...
```
///

### イメージのマウント
(このコンテンツは本書 2.2.10に該当します)

`podman mount`を実行する。そのままではエラーになる。
```
$ podman mount quay.io/rhatdan/myimage
```

(コピペ用)
``` { .yaml .copy }
podman mount quay.io/rhatdan/myimage
```

/// details | 出力結果
    type: success
```
$ podman mount quay.io/rhatdan/myimage
Error: cannot run command "podman mount" in rootless mode, must execute `podman unshare` first
```
///

`podman unshare`を実行します（ローカルにイメージが無い場合は`podman pull quay.io/rhatdan/myimage`を事前に実行します）
`podman unsahre`を実行するとプロンプトが`$`から`#`に変化します。
```
$ podman unshare
# mnt=$(podman image mount quay.io/rhatdan/myimage)
# cat $mnt/var/www/html/index.html
# podman image unmount quay.io/rhatdan/myimage
# exit
```

(コピペ用)
``` { .yaml .copy }
podman unshare
```
``` { .yaml .copy }
mnt=$(podman image mount quay.io/rhatdan/myimage)
```
``` { .yaml .copy }
cat $mnt/var/www/html/index.html
```
``` { .yaml .copy }
podman image unmount quay.io/rhatdan/myimage
```
``` { .yaml .copy }
exit
```

/// details | 出力結果
    type: success
```
$ podman unshare
# mnt=$(podman image mount quay.io/rhatdan/myimage)
# cat $mnt/var/www/html/index.html
<html>
 <head>
 </head>
 <body>
   <h1>Hello World<h1>
 </body>
</html>
# podman image unmount quay.io/rhatdan/myimage
2c7e43d880382561ebae3fa06c7a1442d0da2912786d09ea9baaef87f73c29ae
# exit
exit
```
///

/// admonition | （オプション）`podman image`コマンドの他のサブコマンドの詳細はマニュアルページを参照してください
    type: question
```
$ man podman-image
```
///

/// admonition | まとめ（2.2 コンテナイメージの操作）
    type: abstract
コンテナ実行の元となるコンテナイメージは、コンテナレジストリから入手するだけでなく、名前やタグを付けたものをコンテナレジストリにプッシュできます。
Dockerには無いPodman独自のコマンドとして、`podman image mount`や`podman unshare`があります。これらのコマンドを使用することでより安全にコンテナイメージの中を調査することができます。
///

## 2.3　イメージの構築
### ContainerfileまたはDockerfileのフォーマット
(このコンテンツは本書 2.3.1に該当します)

Containerfileのサンプル
```

```

### アプリケーションのビルドの自動化
(このコンテンツは本書 2.3.2に該当します)

myappディレクトリを作成し、その中にindex.htmlとContainerfileを作成します
```
$ mkdir myapp
$ cat > myapp/index.html << _EOF
<html>
 <head>
 </head>
 <body>
 <h1>Hello World</h1>
 </body>
</html>
_EOF
$ cat > myapp/Containerfile << _EOF
FROM ubi8/httpd-24
COPY index.html /var/www/html/index.html
_EOF
```

出力結果
```
$ mkdir myapp
$ cat > myapp/index.html << _EOF
<html>
 <head>
 </head>
 <body>
 <h1>Hello World</h1>
 </body>
</html>
_EOF
$ cat > myapp/Containerfile << _EOF
FROM ubi8/httpd-24
COPY index.html /var/www/html/index.html
_EOF
```

コンテナイメージをビルドします
```
$ podman build -t quay.io/rhatdan/myimage ./myapp
```

(コピペ用)
``` { .yaml .copy }
podman build -t quay.io/rhatdan/myimage ./myapp
```

/// details | 出力結果
    type: success
```
$ podman build -t quay.io/rhatdan/myimage ./myapp
STEP 1/2: FROM ubi8/httpd-24
STEP 2/2: COPY index.html /var/www/html/index.html
COMMIT quay.io/rhatdan/myimage
--> 1992ba61ae6
Successfully tagged quay.io/rhatdan/myimage:latest
1992ba61ae60d779d865e7de510601d52184f12b25187f6b8c02318ccc240f6b
```
///


コンテナイメージをビルドし、レジストリにプッシュするスクリプトを作成します
/// admonition | 
このシェルスクリプトはコンテナのビルドとプッシュを簡単に自動化するための一例です。
ここではスクリプトの実行は行いませんが、試してみたい方はご自身の`quay.io`アカウントを用意してお試しください。
///
```
$ cat > myapp/automate.sh << _EOF
#!/bin/bash
podman build -t quay.io/rhatdan/myimage ./myapp
podman push quay.io/rhatdan/myimage
_EOF
$ chmod +x myapp/automate.sh
```

(コピペ用)
``` { .yaml .copy }
cat > myapp/automate.sh << _EOF
#!/bin/bash
podman build -t quay.io/rhatdan/myimage ./myapp
podman push quay.io/rhatdan/myimage
_EOF
```
``` { .yaml .copy }
chmod +x myapp/automate.sh
```

/// details | 出力結果
    type: success
```
cat > myapp/automate.sh << _EOF
#!/bin/bash
podman build -t quay.io/rhatdan/myimage ./myapp
podman push quay.io/rhatdan/myimage
_EOF
$ chmod +x myapp/automate.sh
```
///


コンテナイメージのリストを出力する
```
$ podman images
```

(コピペ用)
``` { .yaml .copy }
$ podman images
```

/// details | 出力結果
    type: success
```
$ podman images
REPOSITORY                                TAG         IMAGE ID      CREATED        SIZE
quay.io/rhatdan/myimage                   latest      1992ba61ae60  4 minutes ago  454 MB
registry.access.redhat.com/ubi8/httpd-24  latest      c4127096ce64  2 weeks ago    454 MB
<none>                                    <none>      2c7e43d88038  2 years ago    462 MB
```
///



/// admonition | danglingイメージ
あああ
///

/// admonition | まとめ
    type: abstract
このように...
///