# Chapter 6 ルートレスコンテナ
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

## 6.1 ルートレスPodmanの仕組み
(このコンテンツは本書 6.1に該当します)

環境をクリーンにするためにローカルにあるコンテナイメージを強制的に全て削除します。
```
$ podman rmi --all --force
```

(コピペ用)
``` { .yaml .copy } 
podman rmi --all --force
```

//// details | 出力結果
    type: success
/// admonition | ハンズオンの進行状況により出力内容は変わります
///
```
$ podman rmi --all --force
Untagged: registry.access.redhat.com/ubi8:latest
Untagged: quay.io/podman/stable:latest
Deleted: 4c0f6aace7053de3b9c1476b33c9a763e45a099c8c7ae9117773c9a8e5b8506b
Deleted: 7ad06408d57d20259f8e57797572097ba6c0793f3ea2786805152ae5391d8d9c
```
////




第2章の環境を再構築するために、`podman run`コマンドを実行してコンテナイメージをプルし、`myapp`コンテナを起動します。
```
$ podman run -d -p 8080:8080 --name myapp quay.io/rhatdan/myimage
```

(コピペ用)
``` { .yaml .copy } 
podman run -d -p 8080:8080 --name myapp quay.io/rhatdan/myimage
```

/// details | 出力結果
    type: success
```
$ podman run -d -p 8080:8080 --name myapp quay.io/rhatdan/myimage
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
cb7f4b2cbdc01a4ca1a3cbc3fe9a28b558907caba1fc711419e4cdd2e3682cdd
```
///


### 複数のユーザー識別子(UID)によって所有されるコンテンツが含まれるコンテナイメージ
(このコンテンツは本書 6.1.1に該当します)

`quay.io/rhatdan/myimage`イメージ内の全てのファイルを調べるためにコンテナを起動します。
```
$ podman run --user=root --rm quay.io/rhatdan/myimage -- bash -c "find / -mount -printf \"%U=%u\n\" | sort -un" 2>/dev/null
```

(コピペ用)
``` { .yaml .copy } 
podman run --user=root --rm quay.io/rhatdan/myimage -- bash -c "find / -mount -printf \"%U=%u\n\" | sort -un" 2>/dev/null
```

/// details | 出力結果
    type: success
```
$ podman run --user=root --rm quay.io/rhatdan/myimage -- bash -c "find / -mount -printf \"%U=%u\n\" | sort -un" 2>/dev/null
0=root
48=apache
1001=default
65534=nobody
```
///

#### ユーザー名前空間

Linuxはユーザー名前空間をサポートしており、ホスト上のUID/GIDを名前空間の異なるUID/GIDへとマッピングすることができます。

/// admonition | （オプション）名前空間の詳細な説明は`user_namespaces`のマニュアルページを参照してください
    type: question
```
$ man user_namespaces
```
///


`/etc/subuid`と`/etc/subgid`の中身を確認します。
```
$ cat /etc/subuid
$ cat /etc/subgid
```

(コピペ用)
``` { .yaml .copy } 
cat /etc/subuid
```
``` { .yaml .copy } 
cat /etc/subgid
```

/// details | 出力結果
    type: success
```
$ cat /etc/subuid
myee:100000:65536
ehendricks:165536:65536
rhel:231072:65536
gke-930957db5604c7804fbd:296608:65536
gke-f34473de869e40d6894d:362144:65536
mochtar:427680:65536
$ cat /etc/subgid
myee:100000:65536
ehendricks:165536:65536
rhel:231072:65536
gke-930957db5604c7804fbd:296608:65536
gke-f34473de869e40d6894d:362144:65536
mochtar:427680:65536
```
///


Original
```
$ cat /proc/self/uid_map
```

(コピペ用)
``` { .yaml .copy } 
cat /proc/self/uid_map
```

/// details | 出力結果
    type: success
```
$ cat /proc/self/uid_map
         0          0 4294967295
```
///



Original
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

Original
```
$ podman run --user=root --rm quay.io/rhatdan/myimage -- bash -c "find / -mount -printf \"%U=%u\n\" | sort -un" 2>/dev/null
```

(コピペ用)
``` { .yaml .copy } 
podman run --user=root --rm quay.io/rhatdan/myimage -- bash -c "find / -mount -printf \"%U=%u\n\" | sort -un" 2>/dev/null
```

/// details | 出力結果
    type: success
```
$ podman run --user=root --rm quay.io/rhatdan/myimage -- bash -c "find / -mount -printf \"%U=%u\n\" | sort -un" 2>/dev/null
0=root
48=apache
1001=default
65534=nobody
```
///

ホスト上の`/`ディレクトリの所有者を確認します
```
$ ls -ld /
```

(コピペ用)
``` { .yaml .copy } 
ls -ld /
```

/// details | 出力結果
    type: success
```
$ ls -ld /
dr-xr-xr-x. 18 root root 235 Nov  2  2022 /
```
///


名前空間内の`/`ディレクトリの所有者を確認します
```
$ podman unshare ls -ld /
```

(コピペ用)
``` { .yaml .copy } 
podman unshare ls -ld /
```

/// details | 出力結果
    type: success
```
$ podman unshare ls -ld /
dr-xr-xr-x. 18 nobody nobody 235 Nov  2  2022 /
```
///


Original
```
$ podman unshare bash -c "id ; ls -l /etc/passwd; grep dwalsh /etc/passwd; touch /etc/passwd"
```

(コピペ用)
``` { .yaml .copy } 
podman unshare bash -c "id ; ls -l /etc/passwd; grep dwalsh /etc/passwd; touch /etc/passwd"
```

/// details | 出力結果
    type: success
```
$ podman unshare bash -c "id ; ls -l /etc/passwd; grep dwalsh /etc/passwd; touch /etc/passwd"
uid=0(root) gid=0(root) groups=0(root),65534(nobody) context=system_u:system_r:container_runtime_t:s0
-rw-r--r--. 1 nobody nobody 1808 Oct 16 08:24 /etc/passwd
touch: cannot touch '/etc/passwd': Permission denied
```
///


今度はユーザーのホームディレクトリの所有権をホスト上と名前空間内と比べます。
```
$ ls -ld /home/rhel
```

(コピペ用)
``` { .yaml .copy } 
ls -ld /home/rhel
```

/// details | 出力結果
    type: success
```
$ ls -ld /home/rhel
drwx------. 4 rhel rhel 91 Oct 16 08:28 /home/rhel
```
///

名前空間内では所有権は`root`になります。
```
$ podman unshare ls -ld /home/rhel
```

(コピペ用)
``` { .yaml .copy } 
podman unshare ls -ld /home/rhel
```

/// details | 出力結果
    type: success
```
$ podman unshare ls -ld /home/rhel
drwx------. 4 root root 91 Oct 16 08:28 /home/rhel
```
///


名前空間内でディレクトリを作成し、UIDとGIDを`1:1`に変更します
```
$ podman unshare bash -c "mkdir test;touch test/testfile; chown -R 1:1 test"
```

(コピペ用)
``` { .yaml .copy } 
podman unshare bash -c "mkdir test;touch test/testfile; chown -R 1:1 test"
```

/// admonition | 出力結果はありません

///

ユーザー名前空間の外で所有権を確認すると、この環境では`231072`
```
$ ls -l test
```

(コピペ用)
``` { .yaml .copy } 
ls -l test
```

//// details | 出力結果
    type: success
/// admonition | UID/GIDの番号は実行するホストの環境によって変わることがあります。
///
```
$ ls -l test
total 0
-rw-r--r--. 1 231072 231072 0 Oct 16 08:53 testfile
```
////

`test`ディレクトリを`rm`コマンドで削除してもエラーが出て削除できません。名前空間の外では自身のUIDにのみアクセス権を持つためです。
```
$ rm -rf test
```

(コピペ用)
``` { .yaml .copy } 
rm -rf test
```

/// details | 出力結果
    type: success
```
$ rm -rf test
rm: cannot remove 'test/testfile': Permission denied
```
///


`podman unshare`コマンドで名前空間に入り、`rm`コマンドを実行するとディレクトリの削除ができます。
```
$ podman unshare rm -rf test
```

(コピペ用)
``` { .yaml .copy } 
podman unshare rm -rf test
```

/// admonition | 出力結果はありません

///


/// admonition | （オプション）ユーザー名前空間のケイパビリティの詳細はマニュアルページを参照してください
    type: question
```
$ man capabilities
```
///

#### マウント名前空間

/// admonition | （オプション）マウント名前空間の詳細はマニュアルページを参照してください
    type: question
```
$ man mount_namespaces
```
///


`/proc/self/ns`ディレクトリをリストすると、プロセスが持つ名前空間を確認することができます。
```
$ ls -l /proc/self/ns/user /proc/self/ns/mnt
```

(コピペ用)
``` { .yaml .copy } 
ls -l /proc/self/ns/user /proc/self/ns/mnt
```

/// details | 出力結果
    type: success
```
$ ls -l /proc/self/ns/user /proc/self/ns/mnt
lrwxrwxrwx. 1 rhel rhel 0 Oct 16 14:46 /proc/self/ns/mnt -> 'mnt:[4026531841]'
lrwxrwxrwx. 1 rhel rhel 0 Oct 16 14:46 /proc/self/ns/user -> 'user:[4026531837]'
```
///

`podman unshare`コマンドを使い、名前空間に入った状態で`/proc/self/ns`ディレクトリをリストすると、識別子が変わったことが確認できます。
```
$ podman unshare ls -l /proc/self/ns/user /proc/self/ns/mnt
```

(コピペ用)
``` { .yaml .copy } 
podman unshare ls -l /proc/self/ns/user /proc/self/ns/mnt
```

/// details | 出力結果
    type: success
```
$ podman unshare ls -l /proc/self/ns/user /proc/self/ns/mnt
lrwxrwxrwx. 1 root root 0 Oct 16 14:49 /proc/self/ns/mnt -> 'mnt:[4026532221]'
lrwxrwxrwx. 1 root root 0 Oct 16 14:49 /proc/self/ns/user -> 'user:[4026532220]'
```
///


テストファイルを作成し、`mount`コマンドで`/etc/shadow`ファイルにバインドマウントを行いファイルを書き換える攻撃を想定します。
名前空間の外ではこの試みは失敗します。
```
$ echo hello > /tmp/testfile
$ mount --bind /tmp/testfile /etc/shadow
```

(コピペ用)
``` { .yaml .copy } 
echo hello > /tmp/testfile
```
``` { .yaml .copy } 
mount --bind /tmp/testfile /etc/shadow
```

/// details | 出力結果
    type: success
```
$ echo hello > /tmp/testfile
$ mount --bind /tmp/testfile /etc/shadow
mount: /etc/shadow: must be superuser to use mount.
```
///


名前空間内では名前空間のプロセスは`/etc/shadow`ファイルにバインドマウントできます。
`podman unshare`コマンドを使って名前空間内での挙動を確認します。
```
$ podman unshare bash -c "mount -o bind /tmp/testfile /etc/shadow; cat /etc/shadow"
```

(コピペ用)
``` { .yaml .copy } 
podman unshare bash -c "mount -o bind /tmp/testfile /etc/shadow; cat /etc/shadow"
```

/// details | 出力結果
    type: success
```
$ podman unshare bash -c "mount -o bind /tmp/testfile /etc/shadow; cat /etc/shadow"
hello
```
///

## 6.2 ルートレスPodmanの内部構造
(このコンテンツは本書 6.2に該当します)

Original
```
$ ps -e | grep podman
```

(コピペ用)
``` { .yaml .copy } 
ps -e | grep podman
```

/// details | 出力結果
    type: success
```
$ ps -e | grep podman
   2801 ?        00:00:00 podman pause
```
///

/// admonition | 訳註
    type: info
Podman v4.5.0以降、pauseプロセスの実行に`catatonit`を使用しており、同様の確認を行うためには`ps -e | grep catatonit`を使用します。
https://github.com/containers/podman/pull/12218
///



### ネットワークの設定
(このコンテンツは本書 6.2.3に該当します)

Original
```
$ podman run -d -p 8080:8080 --name myapp registry.access.redhat.com/ubi8/httpd-24
```

### OCIランタイムの起動
(このコンテンツは本書 6.2.5に該当します)

Original
```
$ podman run -d -p 8080:8080 --name myapp registry.access.redhat.com/ubi8/httpd-24
```

### コンテナ化されたアプリケーションの実行が完了するまで
(このコンテンツは本書 6.2.6に該当します)

Original
```
$ podman stop myapp
```