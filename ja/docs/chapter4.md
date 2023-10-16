# Chapter 4　Pod
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

## 4.2 Podの作成
(このコンテンツは本書 4.2に該当します)

事前作業（Chapter 4開始時点の状況に合わせて、下記パターン1 or 2のどちらかの作業を実行してください）

/// admonition | パターン1
Chapter 3の3.1.2でhtmlディレクトリの所有権を変更している場合はhtmlディレクトリとindex.htmlを再作成してください。
///

(コピペ用)
``` { .yaml .copy } 
podman unshare rm -rf html
```
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


/// admonition | パターン2
htmlディレクトリが無い場合は新規でhtmlディレクトリとindex.htmlを作成してください
///

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

PodmanではPodという機能で複数のコンテナを1つのグループとして実行できます。`podman pod create`コマンドを使用してPodを作成できます。

`--name mypod`オプションを使用して`mypod`という名前のPodを作成します。

/// admonition | `--volume`オプションは`-v`でも同様です。Chapter 3を参照ください。
///

```
$ podman pod create -p 8080:8080 --name mypod --volume ./html:/var/www/html:z
```

(コピペ用)
``` { .yaml .copy } 
podman pod create -p 8080:8080 --name mypod --volume ./html:/var/www/html:z
```

/// details | 出力結果
    type: success
```
$ podman pod create -p 8080:8080 --name mypod --volume ./html:/var/www/html:z
b0d68450b3b7ae2d70d8a19de47142af40d0dbd94fecff97b7a1fc3996bbf07e
```
///

/// admonition | （オプション問題）
    type: question
Podをリスト表示する
///

`podman pod ps`コマンドを使用してPodをリスト表示します
```
$ podman pod ps
```

(コピペ用)
``` { .yaml .copy } 
podman pod ps
```

/// details | 出力結果
    type: success
```
$ podman pod ps
POD ID        NAME        STATUS      CREATED         INFRA ID      # OF CONTAINERS
82c5059cc238  mypod       Created     44 seconds ago  7e5b68038395  1
```
///

/// admonition | `podman pod ps`のエイリアス
    type: info
`podman pod ps`の代わりに`podman pod ls`、`podman pod list`使用できます
///


## 4.3 Podへのコンテナ追加
(このコンテンツは本書 4.3に該当します)

`podman create`コマンドを使用して、先ほど作成したPod内にコンテナを作成します。

`--pod mypod`オプションで`mypod`Pod内にコンテナを作成します
```
$ podman create --pod mypod --name myapp quay.io/rhatdan/myimage
```

(コピペ用)
``` { .yaml .copy } 
podman create --pod mypod --name myapp quay.io/rhatdan/myimage
```

/// details | 出力結果
    type: success
```
$ podman create --pod mypod --name myapp quay.io/rhatdan/myimage
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
a0e413d3270111dedca7bdffacc00da362cb01c2ce3d7052efd28356f29a1b2d
```
///

作成したPodにコンテナを追加することができます。まず、シェルスクリプトを実行するサイドカーコンテナの用意をします。

`time.sh`という名前のシェルスクリプトを作成します
```
$ cat > html/time.sh << _EOL
#!/bin/sh
data() {
    echo "<html><head></head><body><h1>"; date;echo "Hello World</h1></body></html>"
    sleep 1
}
while true; do
    data > index.html
done
_EOL
```

(コピペ用)
``` { .yaml .copy } 
cat > html/time.sh << _EOL
#!/bin/sh
data() {
    echo "<html><head></head><body><h1>"; date;echo "Hello World</h1></body></html>"
    sleep 1
}
while true; do
    data > index.html
done
_EOL
```

/// admonition | 出力結果はありません

///

作成した`time.sh`スクリプトに実行権限を付与します
```
$ chmod +x html/time.sh
```

(コピペ用)
``` { .yaml .copy } 
chmod +x html/time.sh
```

/// admonition | 出力結果はありません

///


`time.sh`を実行するサイドカーコンテナをmypodに追加します
```
$ podman create --pod mypod --name time --workdir /var/www/html ubi8 ./time.sh
```

(コピペ用)
``` { .yaml .copy } 
podman create --pod mypod --name time --workdir /var/www/html ubi8 ./time.sh
```

/// details | 出力結果
    type: success
```
$ podman create --pod mypod --name time --workdir /var/www/html ubi8 ./time.sh
Resolved "ubi8" as an alias (/etc/containers/registries.conf.d/001-rhel-shortnames.conf)
Trying to pull registry.access.redhat.com/ubi8:latest...
Getting image source signatures
Checking if image destination supports signatures
Copying blob 70de3d8fc2c6 done  
Copying config 62ac1f7ef5 done  
Writing manifest to image destination
Storing signatures
85e0053dd73cde15758fd1bbe6abe59cd1f8e9ebce318459039d8921db7b2ab5
```
///


## 4.4 Podの起動
(このコンテンツは本書 4.4に該当します)

ここまでに、Podの作成、実行の中心となるプライマリコンテナの追加、サイドカーコンテナの追加を行いました。この時点ではPodに追加したコンテナは停止しています。`podman pod start`コマンドを使ってPodを起動します。

`mypod`Podを起動します
```
$ podman pod start mypod
```

(コピペ用)
``` { .yaml .copy } 
podman pod start mypod
```

/// details | 出力結果
    type: success
```
$ podman pod start mypod
b0d68450b3b7ae2d70d8a19de47142af40d0dbd94fecff97b7a1fc3996bbf07e
```
///


コンテナプロセスを一覧表示します
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
CONTAINER ID  IMAGE                                    COMMAND               CREATED             STATUS         PORTS                   NAMES
b0daeb8cf4ae  localhost/podman-pause:4.4.1-1682527828                        About a minute ago  Up 23 seconds  0.0.0.0:8080->8080/tcp  b0d68450b3b7-infra
a0e413d32701  quay.io/rhatdan/myimage:latest           /usr/bin/run-http...  About a minute ago  Up 23 seconds  0.0.0.0:8080->8080/tcp  myapp
85e0053dd73c  registry.access.redhat.com/ubi8:latest   ./time.sh             35 seconds ago      Up 23 seconds  0.0.0.0:8080->8080/tcp  time
```
///

/// admonition | infraコンテナについて
    type: info
`podman ps`の出力に`localhost/podman-pause`コンテナがあります。これはinfraコンテナと呼ばれ、Podの作成時にPodmanがローカルで自動で作成し実行したものです。ローカルでコンテナを作成する処理が組み込まれているためコンテナイメージをpullする必要はありません。
///



/// admonition | （オプション問題）
    type: question
`podman pod ps`コマンドでPodをリスト表示し、ステータスを確認します。
```
$ podman pod ps
POD ID        NAME        STATUS      CREATED         INFRA ID      # OF CONTAINERS
82c5059cc238  mypod       Running     19 minutes ago  7e5b68038395  3
```
///


## 4.5 Podの停止
(このコンテンツは本書 4.5に該当します)

`podman pod stop`コマンドを使用してPodを停止します。

`mypod`Podを停止します
```
$ podman pod stop mypod
```

(コピペ用)
``` { .yaml .copy } 
podman pod stop mypod
```

/// details | 出力結果
    type: success
```
$ podman pod stop mypod
WARN[0010] StopSignal SIGTERM failed to stop container time in 10 seconds, resorting to SIGKILL 
b0d68450b3b7ae2d70d8a19de47142af40d0dbd94fecff97b7a1fc3996bbf07e
```

`-t`オプションでタイムアウト値を指定しなかったため、デフォルトの10秒の待機後に実行中のコンテナプロセスが停止します
///

コンテナプロセスを表示します
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
CONTAINER ID  IMAGE       COMMAND     CREATED     STATUS      PORTS       NAMES
```
///


## 4.6 Podの表示
(このコンテンツは本書 4.6に該当します)

Podを一覧表示します
```
$ podman pod list
```

(コピペ用)
``` { .yaml .copy } 
podman pod list
```

/// details | 出力結果
    type: success
```
$ podman pod list
POD ID        NAME        STATUS      CREATED        INFRA ID      # OF CONTAINERS
b0d68450b3b7  mypod       Exited      4 minutes ago  b0daeb8cf4ae  3
```
///

## 4.7 Podの削除
(このコンテンツは本書 4.7に該当します)

`podman ps --all`コマンドに`--format`オプションを付けて表示をフィルタリングします。ここでは`ID`、`Image`、`Pod`の3つの要素でフィルタします。
```
$ podman ps --all --format "{{.ID}} {{.Image}} {{.Pod}}"
```

(コピペ用)
``` { .yaml .copy } 
podman ps --all --format "{{.ID}} {{.Image}} {{.Pod}}"
```
/// details | 出力結果
    type: success
```
$ podman ps --all --format "{{.ID}} {{.Image}} {{.Pod}}"
0be17615d860 localhost/podman-pause:4.4.1-1682527828 77b6932c9c44
38e5aa95c038 quay.io/rhatdan/myimage:latest 77b6932c9c44
e2e5f14febab registry.access.redhat.com/ubi8:latest 77b6932c9c44
```
///

`mypod`Podを削除します。
```
$ podman pod rm mypod
```

(コピペ用)
``` { .yaml .copy } 
podman pod rm mypod
```
/// details | 出力結果
    type: success
```
$ podman pod rm mypod
77b6932c9c44581ecd58130cb6a82bc112fee66eac0e01ece5f2908409f02d2a
```
///

Podを一覧表示します
```
$ podman pod ls
```

(コピペ用)
``` { .yaml .copy } 
podman pod ls
```

/// details | 出力結果
    type: success
```
$ podman pod ls
POD ID      NAME        STATUS      CREATED     INFRA ID    # OF CONTAINERS
```
///

`podman ps -a`コマンドに`--format`オプションを付けて表示をフィルタリングします。ここでは`ID`、`Image`の2つの要素でフィルタします。
```
$ podman ps -a --format "{{.ID}} {{.Image}}"
```

(コピペ用)
``` { .yaml .copy } 
podman ps -a --format "{{.ID}} {{.Image}}"
```

/// details | 出力結果
    type: success
```
$ podman ps -a --format "{{.ID}} {{.Image}}"
$
```
///


/// admonition | まとめ
    type: abstract

///