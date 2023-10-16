# Chapter 5 カスタマイズと設定ファイル
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

## 5.1 ストレージの設定ファイル
(このコンテンツは本書 5.1に該当します)

Podmanではファイルシステムのレイヤー、コンテナイメージ、コンテナの保存に`containers/storage`ライブラリを使用します。
`storage.conf`という設定ファイルを使用してこれらのストレージの保存先を設定できます。

`podman info`コマンドで出力をフィルタリングし、Podmanコマンドが使用している`storage.conf`の場所を確認します。
```
$ podman info --format '{{ .Store.ConfigFile }}'
```

(コピペ用)
``` { .yaml .copy } 
podman info --format '{{ .Store.ConfigFile }}'
```

/// details | 出力結果
    type: success
```
$ podman info --format '{{ .Store.ConfigFile }}'
/home/rhel/.config/containers/storage.conf
```
///



### ストレージの場所
(このコンテンツは本書 5.1.1に該当します)



`/etc/containers/storage.conf`を使用してストレージの設定を上書きします。

まず、`sudo`付きでコマンドを実行し、ストレージ設定ファイルのバックアップを行います。

/// admonition | `sudo`のパスワードは`redhat`を入力してください（画面上は入力文字は見えません）
```
...省略...
[sudo] password for rhel: redhat
```
///
```
$ sudo cp /etc/containers/storage.conf /etc/containers/storage.conf.orig
```

(コピペ用)
``` { .yaml .copy } 
sudo cp /etc/containers/storage.conf /etc/containers/storage.conf.orig
```

/// details | 出力結果
    type: success
```
$ sudo cp /etc/containers/storage.conf /etc/containers/storage.conf.orig

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for rhel: 
```
///

ストレージ設定ファイル（`/etc/containers/storage.conf）の変更を行います。
`graphroot = "/var/lib/containers/storage"`となっている箇所を`graphroot = "/var/mystorage"`に変更します。
`sudo`付きで`sed`コマンドを実行します（`vi`コマンドで手動で変更しても構いません）。
```
$ sudo sed -i 's/graphroot = "\/var\/lib\/containers\/storage"/graphroot = "\/var\/mystorage"/' /etc/containers/storage.conf
```

(コピペ用)
``` { .yaml .copy } 
sudo sed -i 's/graphroot = "\/var\/lib\/containers\/storage"/graphroot = "\/var\/mystorage"/' /etc/containers/storage.conf

```

/// admonition | 出力結果はありません

///

ファイルが正常に変更されたか確認します。
```
$ grep -B 1 mystorage /etc/containers/storage.conf
```

(コピペ用)
``` { .yaml .copy } 
grep -B 1 mystorage /etc/containers/storage.conf
```

/// details | 出力結果
    type: success
```
$ grep -B 1 mystorage /etc/containers/storage.conf
# restorecon -R -v /NEWSTORAGEPATH
graphroot = "/var/mystorage"
```
///

ルートモードで`podman info`を実行し、ストレージの設定情報を確認します。`graphRoot:`、`volumePath:`が変更したものになっていることを確認します。
```
$ sudo podman info
```

(コピペ用)
``` { .yaml .copy } 
sudo podman info
```

/// details | 出力結果
    type: success
```
$ sudo podman info
...
store:
  configFile: /etc/containers/storage.conf
...
  graphRoot: /var/mystorage
...
  volumePath: /var/mystorage/volumes
...
```
///


ルートレスモードで`podman info`を実行し、ストレージの設定情報を確認します。`graphRoot:`、`volumePath:`が設定変更前のままであることが確認できます。
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
store:
  configFile: /home/rhel/.config/containers/storage.conf
...
  graphRoot: /home/rhel/.local/share/containers/storage
...
 volumePath: /home/rhel/.local/share/containers/storage/volumes
...
```
///

`/etc/containers/storage.conf`の設定を変更し、`rootless_storage_path`キーを使用してシステム上の全てのユーザーの場所を変更できます。
設定ファイルの該当行のコメントアウトを外し、次のように設定します。`rootless_storage_path = "/var/tmp/$UID/var/mystorage"`

`sudo`付きで`sed`コマンドを実行します（`vi`コマンドで手動で変更しても構いません）。
```
$ sudo sed -i 's/^# rootless_storage_path = "\$HOME\/\.local\/share\/containers\/storage"/rootless_storage_path = "\/var\/tmp\/\$UID\/var\/mystorage"/' /etc/containers/storage.conf
```

(コピペ用)
``` { .yaml .copy } 
sudo sed -i 's/^# rootless_storage_path = "$HOME\/.local\/share\/containers\/storage"/rootless_storage_path = "$HOME\/.local\/share\/containers\/storage"/' /etc/containers/storage.conf
```

/// admonition | 出力結果はありません

///


ファイルが正常に変更されたか確認します。
```
$ grep -B 3 rootless_storage_path /etc/containers/storage.conf
```

(コピペ用)
``` { .yaml .copy } 
grep -B 3 rootless_storage_path /etc/containers/storage.conf
```

/// details | 出力結果
    type: success
```
$ grep -B 3 rootless_storage_path /etc/containers/storage.conf

# Storage path for rootless users
#
rootless_storage_path = "/var/tmp/$UID/var/mystorage"
```
///

ルートレスモードで`podman info`を実行し、ストレージの設定情報を確認します。`configFile:`が元のユーザーのままでありながら、`graphRoot:`、`volumePath:`が変更されていることを確認します。
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
store:
  configFile: /home/rhel/.config/containers/storage.conf
...
 graphRoot: /var/tmp/1002/var/mystorage
...
  volumePath: /var/tmp/1002/var/mystorage/volumes
...
```
///

変更を元に戻すには、バックアップした元ファイルからコピーします。
`sudo`付きで`cp`コマンドを実行します。
```
$ sudo cp /etc/containers/storage.conf.orig /etc/containers/storage.conf
```

(コピペ用)
``` { .yaml .copy } 
sudo cp /etc/containers/storage.conf.orig /etc/containers/storage.conf
```

/// admonition | 出力結果はありません

///

/// admonition | （オプション問題）
    type: question
ルートレスモードで`podman info`を実行し、ストレージの設定情報が元に戻っているか確認しましょう。
///

### ストレージドライバ
(このコンテンツは本書 5.1.2に該当します)

/// admonition | （オプション）`storage.conf`の全てのフィールドの詳細はマニュアルページを参照してください
    type: question
```
$ man containers-storage.conf
```
///


## 5.2 レジストリの設定ファイル
### registries.conf
(このコンテンツは本書 5.2.1に該当します)

Podmanはコンテナイメージのプル、プッシュに`containers/image`ライブラリを使用します。
`registries.conf`という設定ファイルを使用してコンテナレジストリを指定します。また、`policy.json`を使用してコンテナイメージの署名を検証します。

/// admonition | コンテナイメージの署名について
    type: info
本書 11.3 Podmanによるイメージの信頼にて`policy.json`を使用したイメージ署名の設定方法の解説があります。
///

`/etc/containers/registries.conf`を使用してレジストリの設定を変更します。

まず、`sudo`付きでコマンドを実行し、レジストリ設定ファイルのバックアップを行います。
```
$ sudo cp /etc/containers/registries.conf /etc/containers/registries.conf.orig
```

(コピペ用)
``` { .yaml .copy } 
sudo cp /etc/containers/registries.conf /etc/containers/registries.conf.orig
```

/// admonition | 出力結果はありません

///


短縮名でプルする際、事前に解決が用意されたもの以外は表示された選択肢の中からレジストリを選びます。
レジストリ設定ファイルの`unqualified-search-registries`の行の中の`docker.io`を削除し`example.com`を追加します。
`sudo`付きで`sed`コマンドを実行します（`vi`コマンドで手動で変更しても構いません）。
```
$ sudo sed -i 's/unqualified-search-registries = \["registry.access.redhat.com", "registry.redhat.io", "docker.io"\]/unqualified-search-registries = \["registry.access.redhat.com", "registry.redhat.io", "example.com"\]/' /etc/containers/registries.conf
```

(コピペ用)
``` { .yaml .copy } 
sudo sed -i 's/unqualified-search-registries = \["registry.access.redhat.com", "registry.redhat.io", "docker.io"\]/unqualified-search-registries = \["registry.access.redhat.com", "registry.redhat.io", "example.com"\]/' /etc/containers/registries.conf
```

/// admonition | 出力結果はありません

///


ファイルが正常に変更されたか確認します。ルートレスモードで`podman info`を実行し`search:`に`docker.io`の代わりに`example.com`が登録されていることを確認します。
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
  - example.com
...
```
///


短縮名でコンテナイメージをプルしてみます。`foobar`という適当な名前のコンテナイメージを指定して`podman pull`コマンドを実行すると、どのコンテナレジストリからプルするかの選択肢が表示されます。ここで`docker.io`の代わりに`example.com`が選択肢に表示されます。

```
$ podman pull foobar
```

(コピペ用)
``` { .yaml .copy } 
podman pull foobar
```

/// details | 出力結果
    type: success
```
$ podman pull foobar
? Please select an image: 
  ▸ registry.access.redhat.com/foobar:latest
    registry.redhat.io/foobar:latest
    example.com/foobar:latest
```

プルを強制的に停止するには ++ctrl+c++ を押します

///


変更を元に戻すには、バックアップした元ファイルからコピーします。
`sudo`付きで`cp`コマンドを実行します。
```
$ sudo cp /etc/containers/registries.conf.orig /etc/containers/registries.conf
```

(コピペ用)
``` { .yaml .copy } 
sudo cp /etc/containers/registries.conf.orig /etc/containers/registries.conf
```

/// admonition | 出力結果はありません

///

/// admonition | （オプション問題）
    type: question
ルートレスモードで`podman info`を実行し、レジストリの設定情報が元に戻っているか確認しましょう。また`podman pull foobar`コマンドを実行して`docker.io`が選択肢に表示されるか確認しましょう。
///

#### コンテナレジストリからのプルをブロックする
レジストリ設定ファイルの変更を行います。
`sudo`付きで`sed`コマンドを実行します（`vi`コマンドで手動で変更しても構いません）。
```
$ sudo sed -i 's/# \[\[registry\]\]/\[\[registry\]\]\nLocation = "docker.io"\nblocked=true/' /etc/containers/registries.conf
```

ファイルが正常に変更されたか確認します。ルートレスモードで`podman info`を実行し`registries:`に`docker.io`が`Blocked: true`になっていることを確認します。
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
  docker.io:
    Blocked: true
...
```
///


`docker.io`からコンテナイメージをプルし、実行がブロックされることを確認します。
```
$ podman pull docker.io/ubuntu
```

(コピペ用)
``` { .yaml .copy } 
podman pull docker.io/ubuntu
```

/// details | 出力結果
    type: success
```
$ podman pull docker.io/ubuntu
Trying to pull docker.io/library/ubuntu:latest...
Error: initializing source docker://ubuntu:latest: registry docker.io is blocked in /etc/containers/registries.conf or /home/rhel/.config/containers/registries.conf.d
```
///

変更を元に戻すには、バックアップした元ファイルからコピーします。
`sudo`付きで`cp`コマンドを実行します。
```
$ sudo cp /etc/containers/registries.conf.orig /etc/containers/registries.conf
```

(コピペ用)
``` { .yaml .copy } 
sudo cp /etc/containers/registries.conf.orig /etc/containers/registries.conf
```

/// admonition | 出力結果はありません

///


インターネット隔離環境で、インターネット上のコンテナレジストリにアクセスできない状況があります。内部のコンテナレジストリにインターネット上からミラーしたコンテナイメージを置き、同じコマンドでイメージをプルして内部のコンテナレジストリからイメージを入手することができます。

レジストリ設定ファイルを次のように変更します。
```

```

レジストリ設定ファイルの変更を行います。
`sudo`付きで`sed`コマンドを実行します（`vi`コマンドで手動で変更しても構いません）。


```
$ sudo sed -i '/# \[\[registry\]\]/s/# \[\[registry\]\]/\[\[registry\]\]\nlocation="registry.access.redhat.com"\n[[registry.mirror]]\nlocation="mirror-1.com"/' /etc/containers/registries.conf
```

(コピペ用)
``` { .yaml .copy } 
sudo sed -i '/# \[\[registry\]\]/s/# \[\[registry\]\]/\[\[registry\]\]\nlocation="registry.access.redhat.com"\n[[registry.mirror]]\nlocation="mirror-1.com"/' /etc/containers/registries.conf
```

/// admonition | 出力結果はありません

///

```
$ podman pull registry.access.redhat.com/ubi8/httpd-24:latest
```

/// admonition | （オプション）`registries.conf`の全てのフィールドの詳細はマニュアルページを参照してください
    type: question
```
$ man containers-registries.conf
```
///


## 5.3 エンジンの設定ファイル
(このコンテンツは本書 5.3に該当します)

Podmanで実行する全てのコンテナに同じ環境変数をします。
`podman run`コマンドでUBI 8のコンテナイメージを実行し、デフォルトの環境変数を表示します。`--rm`オプションを付けているのでコンテナが起動し、`printenv`コマンドが実行された後コンテナは停止とともの削除されます。
```
$ podman run --rm ubi8 printenv
```

(コピペ用)
``` { .yaml .copy } 
podman run --rm ubi8 printenv
```

//// details | 出力結果
    type: success
/// admonition
    type: info
`HOSTNAME`はコンテナの実行の度にランダムなものが設定されます
///
```
$ podman run --rm ubi8 printenv
Resolved "ubi8" as an alias (/etc/containers/registries.conf.d/000-shortnames.conf)
Trying to pull registry.access.redhat.com/ubi8:latest...
Getting image source signatures
Checking if image destination supports signatures
Copying blob sha256:a0534d4abf263a5a894b1468dc1e3c785b1e9b29a9e78c8a5e8a99b4f1badb27
Copying config sha256:8adef039dbf2331ae2dc8cba0742b499ba11c7b0f9fea33a996d9751fb2f4baf
Writing manifest to image destination
Storing signatures
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
container=oci
HOME=/root
HOSTNAME=09f826169179
```
////


ルートレスモードでの実行に適用するため、ユーザーのホームディレクトリに`env.conf`ファイルを作成します。
環境変数`foo=bar`を設定するため、設定ファイルに`env=[ "foo=bar" ]`を記述します。

/// admonition | `env.conf`について
    type: info
実際には`containers.conf.d`にある`*.conf`ファイルが読み込まれます。ファイル名は`env.conf`でなくても構いません。
///
```
$ mkdir -p $HOME/.config/containers/containers.conf.d
$ cat << _EOF > $HOME/.config/containers/containers.conf.d/env.conf
[containers]
env=[ "foo=bar" ]
_EOF
```

(コピペ用)
``` { .yaml .copy } 
mkdir -p $HOME/.config/containers/containers.conf.d
```
``` { .yaml .copy } 
cat << _EOF > $HOME/.config/containers/containers.conf.d/env.conf
[containers]
env=[ "foo=bar" ]
_EOF
```

/// admonition | 出力結果はありません

///

再度`podman run`コマンドで環境変数を表示すると`foo=bar`が設定されていることが確認できます。
```
$ podman run --rm ubi8 printenv
```

(コピペ用)
``` { .yaml .copy } 
podman run --rm ubi8 printenv
```

/// details | 出力結果
    type: success
```
$ podman run --rm ubi8 printenv
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
TERM=xterm
container=oci
foo=bar
HOME=/root
HOSTNAME=0b22561c7753
```
///




Podmanの設定ファイルは`containers.conf`を使用します。
PodmanをCI/CDシステムで実行したり、各Linuxディストリビューションが提供するよりも新しいバージョンのPodmanを試したい場合などにおいて、**Podmanをコンテナで実行する**ニーズがあります（Podman in Podmanとも言う）。
その際に使用できるのがPodmanの公式コンテナイメージ`quay.io/podman/stable`です。
Podmanをコンテナ内で実行するために適切な設定を行うために`containers.conf`を利用して設定が行われています。

`podman/stable`コンテナイメージを使ってコンテナを実行し、`containers.conf`の内容を参照します。
```
$ podman run quay.io/podman/stable cat /etc/containers/containers.conf
```

(コピペ用)
``` { .yaml .copy } 
podman run quay.io/podman/stable cat /etc/containers/containers.conf
```

/// details | 出力結果
    type: success
```
$ podman run quay.io/podman/stable cat /etc/containers/containers.conf
Trying to pull quay.io/podman/stable:latest...
Getting image source signatures
Copying blob 45055f372b99 done  
Copying blob b30887322388 done  
Copying blob 7978d18623f4 done  
Copying blob a6e2d067da71 done  
Copying blob 584c4c05fc10 done  
Copying blob 383f5c3bb112 done  
Copying blob c0c0aed1fa32 done  
Copying blob aa2f3b2edf32 done  
Copying config 7ad06408d5 done  
Writing manifest to image destination
Storing signatures
[containers]
netns="host"
userns="host"
ipcns="host"
utsns="host"
cgroupns="host"
cgroups="disabled"
log_driver = "k8s-file"
[engine]
cgroup_manager = "cgroupfs"
events_logger="file"
runtime="crun"
```
///


`podman/stable`コンテナイメージを使ってコンテナ内でコンテナを実行します。
```
$ podman run --security-opt label=disable --device /dev/fuse --user podman quay.io/podman/stable podman run ubi8-micro echo hi
```

(コピペ用)
``` { .yaml .copy } 
podman run --security-opt label=disable --device /dev/fuse --user podman quay.io/podman/stable podman run ubi8-micro echo hi
```

/// details | 出力結果
    type: success
```
$ podman run --security-opt label=disable --device /dev/fuse --user podman quay.io/podman/stable podman run ubi8-micro echo hi
Resolved "ubi8-micro" as an alias (/etc/containers/registries.conf.d/000-shortnames.conf)
Trying to pull registry.access.redhat.com/ubi8-micro:latest...
Getting image source signatures
Checking if image destination supports signatures
Copying blob sha256:65c9e8ba90a0ebb7228c51a4e3fdaa4a759ab28d24446b84760bd8fc9890d585
Copying config sha256:19a567442b093c27c72d9689b5b9cf9c1a42eb8c9bb151ba4e00af324052edb8
Writing manifest to image destination
Storing signatures
hi
```
///


/// admonition | （オプション）`containers.conf`の全てのフィールドの詳細はマニュアルページを参照してください
    type: question
```
$ man containers.conf
```
///

/// admonition | まとめ
    type: abstract
Podmanの設定を行う各種ファイルと、設定ファイルに設定を行った時の動作について学びました。
ストレージの設定ファイル`storage.conf`、レジストリの設定ファイル`registries.conf`、コンテナエンジンの設定ファイル`containers.conf`、それぞれに設定を行うことでPodmanの動作を詳細にカスタマイズできます。Podman公式から`podman/stable`コンテナイメージも提供されています。このコンテナイメージを使ってコンテナ内でコンテナを実行する「Podman in Podman」も実現できます。
///