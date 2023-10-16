# Chapter 3　ボリューム
- レベル:初級
- 見込み時間:15分
- コンテンツ


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
## 3.1 コンテナでのボリュームの使用

### コンテナでのボリュームの使用
(このコンテンツは本書 3.1に該当します)

htmlディレクトリを作成し、その中にindex.htmlファイルを作成します
```
$ mkdir html
$ cat > html/index.html << _EOF
<html>
 <head>
 </head>
 <body>
  <h1>Goodbye World</h1>
 </body>
</html>
_EOF
```

(コピペ用)
``` { .yaml .copy } 
mkdir html
```

``` { .yaml .copy } 
cat > html/index.html << _EOF
<html>
 <head>
 </head>
 <body>
  <h1>Goodbye World</h1>
 </body>
</html>
_EOF
```

/// admonition | 出力結果はありません

///

`podman run`コマンド実行時に`-v`オプションを使用することで、ホスト上のコンテンツをコンテナ内にマウントすることができます。

`-v ./html:/var/www/html:ro,z`オプションを使用してコンテナを起動します

/// admonition | `:ro,z`オプションについて
    type: info
`:ro`オプションはボリュームをコンテナ内で読み取り専用モードでマウントします。
`:z`オプションはSELinuxのラベルを、コンテナが読み書きできるように再ラベル付けします。詳細は本書3.1.2内の`SELinux のボリュームオプション`を参照ください。
///


```
$ podman run -d -v ./html:/var/www/html:ro,z -p 8080:8080 quay.io/rhatdan/myimage
```

(コピペ用)
``` { .yaml .copy } 
podman run -d -v ./html:/var/www/html:ro,z -p 8080:8080 quay.io/rhatdan/myimage
```

/// details | 出力結果
    type: success
```
$ podman run -d -v ./html:/var/www/html:ro,z -p 8080:8080 quay.io/rhatdan/myimage
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
654cf2e2a6382f901b775296691cdf47231efd7da6c1bdb877f00ef9c4d6a654
```
///


`curl`コマンドでアクセスして確認します
```
$ curl localhost:8080
```

(コピペ用)
``` { .yaml .copy } 
curl localhost:8080
```

/// details | 出力結果
    type: success
```
$ curl localhost:8080
<html>
 <head>
 </head>
 <body>
 <h1>Goodbye World</h1>
 </body>
</html>
```
///

<!--
- ブラウザで確認
-->


`podman rm`コマンドを使用して実行中のコンテナを削除します。コンテナの削除後もコンテナ内にマウントしたコンテンツは削除されません。

/// admonition | `--latest`、`--force`オプションについて
    type: info
`podman rm`コマンドに合わせて`--latest`、`--force`オプションを付けることで、少ないコマンドで実行中のコンテナを強制的に削除できます。

`--latest`または`-l`はコンテナ名やIDの代わりに最後に作成されたコンテナを使用します。
`--force`または`-f`は実行中または一時停止中のコンテナを強制的に削除します。

///

```
$ podman rm --latest --force
```

(コピペ用)
``` { .yaml .copy } 
podman rm --latest --force
```

/// details | 出力結果
    type: success
```
$ podman rm --latest --force
654cf2e2a6382f901b775296691cdf47231efd7da6c1bdb877f00ef9c4d6a654
```
///


/// admonition | （オプション問題）
    type: question
htmlディレクトリの中を確認し、コンテンツが削除されていないことを確認します
///

```
$ ls -l html
```

(コピペ用)
``` { .yaml .copy } 
ls -l html
```

/// details | 出力結果
    type: success
```
$ ls -l html

```
///


htmlディレクトリを削除します
```
$ rm -rf html
```

(コピペ用)
``` { .yaml .copy } 
rm -rf html
```

/// admonition | 出力結果はありません

///

### 名前付きボリューム
(このコンテンツは本書 3.1.1に該当します)

Podmanのコンテナでデータを永続化する別の仕組みとして「名前付きボリューム（または単にボリュームと呼ぶ）」があります。
名前付きボリュームは`podman volume create`コマンドで作成できます。名前付きボリュームの実体はホスト上の特定のパス（ローカルコンテナストレージ）にあります。

`podman volume create`コマンドを使用してボリューム`webdata`を作成します
```
$ podman volume create webdata
```

(コピペ用)
``` { .yaml .copy } 
podman volume create webdata
```

/// details | 出力結果
    type: success
```
$ podman volume create webdata
webdata
```
///


ボリュームの情報を確認します
```
$ podman volume inspect webdata
```

(コピペ用)
``` { .yaml .copy } 
podman volume inspect webdata
```

/// details | 出力結果
    type: success
```
$ podman volume inspect webdata
[
     {
          "Name": "webdata",
          "Driver": "local",
          "Mountpoint": "/home/rhel/.local/share/containers/storage/volumes/webdata/_data",
          "CreatedAt": "2023-09-11T15:55:56.024196806Z",
          "Labels": {},
          "Scope": "local",
          "Options": {},
          "MountCount": 0,
          "NeedsCopyUp": true,
          "NeedsChown": true
     }
]
```
///


ボリューム内にコンテンツを格納するために、ローカルコンテナストレージ上のディレクトリにコンテンツを作成します。
/// admonition | ホスト上で名前付きボリュームにコンテンツを作成する
この例では、`/home/rhel/.local/share/containers/storage/volumes/webdata/_data/`にコンテンツを作成することで、ボリューム内にコンテンツを作成したことと同等の結果を得られます。
///

```
$ cat > /home/rhel/.local/share/containers/storage/volumes/webdata/_data/index.html << _EOL
<html>
 <head>
 </head>
 <body>
 <h1>Goodbye World</h1>
 </body>
</html>
_EOL
```

(コピペ用)
``` { .yaml .copy } 
cat > /home/rhel/.local/share/containers/storage/volumes/webdata/_data/index.html << _EOL
<html>
 <head>
 </head>
 <body>
 <h1>Goodbye World</h1>
 </body>
</html>
_EOL
```

/// admonition | 出力結果はありません

///

`-v`オプションでホスト上のコンテンツをコンテナにマウントしたのと同様に、名前付きボリュームをコンテナにマウントすることができます。


`-v`オプションで名前付きボリューム`webdata`を使用してコンテナを作成します
```
$ podman run -d -v webdata:/var/www/html:ro,z -p 8080:8080 quay.io/rhatdan/myimage
```
(コピペ用)
``` { .yaml .copy } 
podman run -d -v webdata:/var/www/html:ro,z -p 8080:8080 quay.io/rhatdan/myimage
```

/// details | 出力結果
    type: success
```
$ podman run -d -v webdata:/var/www/html:ro,z -p 8080:8080 quay.io/rhatdan/myimage
8c1c69967f9cef8a0de312f65cb0e70c0be2e6af3965d299aec75e8f39d6e25a
```
///

/// admonition | （オプション問題）
    type: question
`curl`コマンドやブラウザを使ってボリュームに作成したコンテンツ（`Goodbye World`が表示される）にアクセスできるか確認しましょう
///

`podman stop`コマンドを使用して実行中のコンテナを一時停止します。

`-t 0`オプションとコンテナIDを指定してコンテナを停止します。コンテナIDはご自身の環境のものを指定してください。

/// admonition | `-t 0`オプションについて
    type: info
`--time`または`-t`オプションは、強制的に停止するまでの待機時間を秒数で指定します。`-t 0`の場合は即座に停止が実行されます。
///



/// admonition | コンテナIDについて
    type: info
コンテナIDはコンテナ作成時にランダムに割り当てられる64文字のハッシュ値です。このハッシュ値の先頭12文字をコンテナIDとして指定することができます。`podman ps`コマンドを使用してコンテナIDを確認できます。
///

```
$ podman stop -t 0 8c1c69967f9c
```

(コピペ用)
``` { .yaml .copy } 
podman stop -t 0 8c1c69967f9c
```

/// details | 出力結果
    type: success
```
$ podman stop -t 0 8c1c69967f9c
8c1c69967f9c
```
///

作成したボリュームは`podman volume rm`コマンドを使って削除できます。

`--force`オプションとボリューム名を指定して強制的にボリュームの削除とコンテナの削除を行います。

/// admonition | `--force`オプションの使用について
    type: info
`podman volume rm`のみを使用した場合、今回はコンテナがまだ停止状態で残っているためエラーが出て削除ができません。
```
Error: volume webdata is being used by the following container(s): e55377526ab98a9a875c244c9f110b70ed82656f8c3e6d6456181ed63594fbd1: volume is being used
```
そのため、ボリュームとコンテナを同時に強制的に削除するために`--force`オプションを使用します。
///

```
$ podman volume rm --force webdata
```

(コピペ用)
``` { .yaml .copy } 
podman volume rm --force webdata
```

/// details | 出力結果
    type: success
```
$ podman volume rm --force webdata
webdata
```
///


`podman volume list`を実行して、ボリュームが存在しないことを確認します
```
$ podman volume list
```

(コピペ用)
``` { .yaml .copy } 
podman volume list
```

/// details | 出力結果
    type: success
```
$ podman volume list
$
```
///

/// admonition | （オプション問題）
    type: question
`podman ps -a`コマンドを使ってコンテナが削除されているか確認しましょう
///


ボリュームが存在しない状態で`podman run`を実行できます。その場合、ボリュームは自動的に作成されます。

`webdata1`ボリュームを指定してコンテナを起動します。そしてボリュームが作成されているか確認します。
```
$ podman run -d -v webdata1:/var/www/html:ro,z -p 8080:8080 quay.io/rhatdan/myimage
$ podman volume list
```

(コピペ用)
``` { .yaml .copy } 
podman run -d -v webdata1:/var/www/html:ro,z -p 8080:8080 quay.io/rhatdan/myimage
```
``` { .yaml .copy } 
podman volume list
```

/// details | 出力結果
    type: success
```
$ podman run -d -v webdata1:/var/www/html:ro,z -p 8080:8080 quay.io/rhatdan/myimage
b2613ca208811ac992a1215cffb9a20b5eb97e0d5cb543aacfe6616fc5bb6e49
$ podman volume list
DRIVER      VOLUME NAME
local       webdata1
```
///

/// admonition | （オプション問題）
    type: question
`curl`コマンドやブラウザを使ってコンテンツにアクセスできるか確認しましょう。またその際に表示される内容についても確認しましょう。
///


/// admonition | 空のボリュームをマウントした際の挙動
`webdata1`ボリュームは空なのでコンテナイメージ内の元のindex.htmlは保持された状態になります。
コンテナイメージは階層構造を持ち、PodmanはLinuxカーネルのoverlayfs機能を使いイメージが重ね合わされた結果のファイルシステムを実行します。
今回のようにボリュームが空で同名のファイルが存在しない場合は元のデータは上書きされません。
///



ボリュームを強制的に削除します。また同時にコンテナも削除します。
```
$ podman volume rm --force webdata1
```

(コピペ用)
``` { .yaml .copy } 
podman volume rm --force webdata1
```

/// details | 出力結果
    type: success
```
$ podman volume rm --force webdata1
webdata1
```
///


/// admonition | （オプション問題）
    type: question
`podman volume list`、`podman ps`コマンドを使ってコンテナが削除されているか確認しましょう
///


### ボリュームのマウントオプション
(このコンテンツは本書 3.1.2に該当します)

これまでにボリュームマウント時に使用した`:ro`や`:z`オプション以外にもPodmanでは便利なオプションがあります。

#### U ボリュームオプション
`:U`オプションはその一つです。ルートレスコンテナにおける所有権の問題を簡単に解決できるオプションです。ルートレスコンテナで、コンテナ内のプロセスが`UID==60`として実行する例を考えてみましょう。

ルートレスコンテナにおけるユーザー名前空間マッピングを確認します。
```
$ podman unshare cat /proc/self/uid_map
```

(コピペ用)
``` { .yaml .copy } 
podman unshare cat /proc/self/uid_map
```

/// details | 出力結果
    type: success
```
$ podman unshare cat /proc/self/uid_map
         0       1002          1
         1     231072      65536
```
///

この結果は下記を表します

- コンテナ内の`UID 0`はホスト上の`UID 1002`にマップされます。
- `UID 1`は`UID 231072`、`UID 2`は`UID 231073` ... `UID 65536`は`UID 296607`とマップされます。

コンテナ内で`UID 60`として実行するプロセスがボリュームに書き込む場合、ホスト上のユーザーの所有権と異なるため書き込みができません。
ユーザー名前空間内でボリュームの所有権を`UID 60`にするために、`podman unshare`を使用して`chown`を実行します。


/// admonition | `podman unshare`コマンドについて
    type: info
`podman unshare`コマンドはコンテナを実行せずにユーザー名前空間内に入ることができます。このコマンドはPodmanの独自のコマンドでDockerにはありません。
///


```
$ podman unshare chown 60:60 ./html
```

(コピペ用)
``` { .yaml .copy } 
podman unshare chown 60:60 ./html
```

/// admonition | 出力結果はありません

///

/// admonition | （オプション問題）
    type: question
`ls -ld`コマンドでhtmlディレクトリの所有権がどのようになったか確認してみましょう
```
$ ls -ld
drwxr-xr-x. 2 231131 231131 6 Oct 14 14:58 html
```
///

コンテナイメージによってはコンテナ内で実行するプロセスが特定のUIDを指定しているものがあります。mariadbコンテナイメージはその一つで、データベースのプロセスはmysqlユーザ（UID==999）によって実行されます。

mariadbコンテナイメージのmysqlユーザーのUIDを確認します。

```
$ podman run docker.io/mariadb grep mysql /etc/passwd
```

(コピペ用)
``` { .yaml .copy } 
podman run docker.io/mariadb grep mysql /etc/passwd
```

/// details | 出力結果
    type: success
```
$ podman run docker.io/mariadb grep mysql /etc/passwd
Trying to pull docker.io/library/mariadb:latest...
Getting image source signatures
Copying blob 50ee0c078c93 done  
Copying blob 43f89b94cd7d done  
Copying blob dfb413a01c7e done  
Copying blob 1d3f76b535d3 done  
Copying blob f7efd05ec01e done  
Copying blob fe2ff83c75df done  
Copying blob 6975e72928bb done  
Copying blob 561d1b426cbd done  
Copying config f35870862d done  
Writing manifest to image destination
Storing signatures
mysql:x:999:999::/home/mysql:/bin/sh
```
///

/// admonition | `:U`オプションについて
    type: info
コンテナイメージによってはコンテナ内で実行するプロセスが特定のUIDを指定しているものがあります。
///


データベース用のディレクトリを作成します。ディレクトリの所有権がホスト上のユーザー（rhel）であることを確認します。
```
$ mkdir mariadb
$ ls -ld mariadb/
```

(コピペ用)
``` { .yaml .copy } 
mkdir mariadb
```
``` { .yaml .copy } 
ls -ld mariadb/
```

/// details | 出力結果
    type: success
```
$ mkdir mariadb
$ ls -ld mariadb/
drwxr-xr-x. 2 rhel rhel 6 Oct 14 15:25 mariadb/
```
///

mariadbコンテナイメージを実行します。

`:U`オプションを使用し`mariadb`ディレクトリをコンテナ内の`/var/lib/mariadb`ディレクトリにマウントします。また`--user mysql`オプションを使用し、コンテナの実行ユーザーを`mysql`と指定します。
```
$ podman run --user mysql -v ./mariadb:/var/lib/mariadb:U docker.io/mariadb ls -ld /var/lib/mariadb
```

(コピペ用)
``` { .yaml .copy } 
podman run --user mysql -v ./mariadb:/var/lib/mariadb:U docker.io/mariadb ls -ld /var/lib/mariadb
```

/// details | 出力結果
    type: success
```
$ podman run --user mysql -v ./mariadb:/var/lib/mariadb:U docker.io/mariadb ls -ld /var/lib/mariadb
drwxr-xr-x. 2 mysql mysql 6 Oct 14 15:25 /var/lib/mariadb
```
///

<!-- 
#### SELinux のボリュームオプション
```
$ podman run --security-opt label=disable -v /home/rhel:/home/rhel -p 8080:8080 quay.io/rhatdan/myimage
```

```
podman run --security-opt label=disable -v /home/rhel:/home/rhel -p 8080:8080 quay.io/rhatdan/myimage
```

/// details | 出力結果
    type: success
```
```
///
-->

### podman run --mountコマンドオプション
(このコンテンツは本書 3.1.3に該当します)

/// admonition | （オプション）`podman run --mount`コマンドでサポートされているマウントタイプや`podman volume`のサブコマンドの詳細はマニュアルページを参照してください
    type: question
```
$ man podman-mount
$ podman-volume-create
$ podman-volume-exists
$ podman-volume-export
$ podman-volume-import
$ podman-volume-inspect
$ podman-volume-list
$ podman-volume-prune
$ podman-volume-rm
```
///


/// admonition | まとめ
    type: abstract
バインドマウントや名前付きボリュームを使用することで永続化したデータをコンテナで使用することができます。
ボリュームはホスト上のファイルシステムをコンテナ内にマウントするために、SELinuxやユーザー名前空間のセキュリティ上の問題に対処する必要があります。
Podmanではそのための便利なボリュームマウントオプションがあります。
///