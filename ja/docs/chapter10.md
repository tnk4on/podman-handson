# Chapter 10 コンテナ隔離におけるセキュリティ
- レベル:初級
- 見込み時間:15分
- コンテンツ

## 読み取り専用のLinuxカーネル擬似ファイルシステム
### マスクされたパスの解除
(このコンテンツはChapter 10.1.1に該当します)

Original
```
$ podman run --rm ubi8 ls /proc/scsi
$ podman run --rm --security-opt unmask=/proc/scsi ubi8 ls /proc/scsi
$ podman run --rm --security-opt unmask=/proc/* ubi8 ls /proc/scsi
$ man podman run
```


### マスクされるパスの追加
(このコンテンツはChapter 10.1.2に該当します)

Original
```
$ podman run --rm ubi8 ls /proc/sys/dev
$ podman run --rm --security-opt mask=/proc/sys/dev ubi8 ls /proc/sys/dev
$ podman run --rm ubi8 cat /proc/self/mountinfo
```


## Linuxケイパビリティ
(このコンテンツはChapter 10.2に該当します)

Original
```
$ capsh --print
```

### Linuxケイパビリティの削除
(このコンテンツはChapter 10.2.1に該当します)

Original
```
$ podman run --rm ubi8 capsh --print
```

### ケイパビリティの削除
(このコンテンツはChapter 10.2.3に該当します)

Original
```
$ podman run --cap-drop CAP_NET_BIND_SERVICE ubi8 capsh --print
```

Original
```
$ podman run --cap-drop all ubi8 capsh --print
```

### ケイパビリティの追加
(このコンテンツはChapter 10.2.4に該当します)

Original
```
$ podman run --cap-add CAP_NET_RAW ubi8 capsh --print
```

Original
```
$ podman run --cap-drop=all --cap-add CAP_NET_RAW ubi8 capsh --print
```


## 10.3 ユーザー名前空間によるUIDの分離
### --userns=autoフラグを利用したコンテナの隔離
(このコンテンツはChapter 10.3.1に該当します)

Original
```
# cat /etc/subuid
# cat /etc/subgid
```

Original
```
# podman run --userns=auto ubi8 cat /proc/self/uid_map
# podman run --user=2000 --userns=auto ubi8 cat /proc/self/uid_map
# podman run --userns=auto:size=5000 ubi8 cat /proc/self/uid_map
# podman run --rm --userns=auto ubi8 cat /proc/self/uid_map
# podman run --rm --userns=auto ubi8 cat /proc/self/uid_map
```


### ユーザー名前空間内のLinuxのケイパビリティ
(このコンテンツはChapter 10.3.2に該当します)

Original
```
# podman run --rm ubi8 capsh --print | grep Current
# podman run --rm --userns=auto ubi8 capsh --print | grep Current
# podman run --rm --userns=auto:size=5000 ubi8 chown 6000 /etc/motd
# podman run --rm --userns=auto:size=5000 ubi8 chown 4000 /etc/motd
```


### ルートレスPodmanにおける--userns=autoフラグ
(このコンテンツはChapter 10.3.3に該当します)

Original
```
$ podman run --userns=auto ubi8 cat /proc/self/uid_map
$ podman run --userns=auto ubi8 cat /proc/self/uid_map
$ podman run --rm ubi8 cat /proc/self/uid_map
```

Original
```
# mkdir /mnt/test
# ls -ld /mnt/test
# podman run --rm -v /mnt/test:/mnt/test --userns=auto ubi8 ls -ld /mnt/test
# podman run --rm -v /mnt/test:/mnt/test:Z --userns=auto ubi8 touch /mnt/test/test1
```

### ユーザーボリュームにおける--userns=autoフラグ
(このコンテンツはChapter 10.3.4に該当します)

Original
```
# ls -ld /mnt/test
# podman run --rm -v /mnt/test:/mnt/test:Z,U --userns=auto ubi8 touch /mnt/test/test1
# ls -ld /mnt/test
# chown -R root:root /mnt/test
# podman run --rm -v /mnt/test:/mnt/test:idmap,Z --userns=auto ubi8 ls -ld /mnt/test
# podman run --rm -v /mnt/test:/mnt/test:idmap,Z --userns=auto ubi8 touch /mnt/test/test
# ls -l /mnt/test
```

## 10.4 PID名前空間によるプロセスの分離
(このコンテンツはChapter 10.4に該当します)

Original
```
$ podman run --rm ubi8 find /proc -maxdepth 1 -type d -regex ".*/[0-9]*"
```

## 10.5 ネットワーク名前空間によるネットワークの分離
(このコンテンツはChapter 10.5に該当します)

Original
```
$ podman network create net1
$ podman network create net2
```

Original
```
$ podman run -d --network net1 --name cnet1 ubi8 sleep 1000
$ podman run --network net1 alpine ping -c 1 cnet1
```

Original
```
$ podman run --rm alpine ping -c 1 cnet1
$ podman run alpine ping -c 1 10.89.0.4
$ podman run --rm --network net2 alpine ping -c 1 cnet1
```

## 10.6 IPC名前空間によるIPCの分離
(このコンテンツはChapter 10.6に該当します)

Original
```
$ podman run -d --rm --name ipc1 ubi8 bash -c "touch /dev/shm/ipc1; sleep 1000"
$ podman run --rm ubi8 ls /dev/shm
$ podman run --rm --ipc=container:ipc1 ubi8 ls /dev/shm
```

## 10.8 SELinuxによるファイルシステムの分離
### SELinux type enforcement
(このコンテンツはChapter 10.8.1に該当します)

Original
```
$ podman run --rm ubi8 cat /proc/self/attr/current
$ podman run --rm --privileged ubi8 cat /proc/self/attr/current
$ podman run --rm ubi8 ls -Z /
$ ls -1Z $HOME/.ssh/
$ podman run -v $HOME/.ssh:/.ssh ubi8 ls /.ssh
```

Original
```
$ mkdir foo
$ ls -Zd foo
$ podman run -v ./foo:/foo ubi8 touch /foo/bar
$ podman run --privileged -v ./foo:/foo ubi8 touch /foo/bar
$ ls -Z foo
$ rm foo/bar
$ podman run -v ./foo:/foo:Z ubi8 touch /foo/bar
$ ls -Z ./foo
```

### SELinuxのMulti-Category Securityによる分離
(このコンテンツはChapter 10.8.2に該当します)

Original
```
$ podman run --rm ubi8 cat /proc/self/attr/current
$ podman run --rm ubi8 cat /proc/self/attr/current
```

Original
```
$ ls -Z ./foo
$ podman run -v ./foo:/foo ubi8 touch /foo/bar
$ podman run --security-opt label=level:s0:c454,c510 -v ./foo:/foo ubi8 touch /foo/bar
```

Original
```
$ podman run -v ./foo:/foo:z ubi8 touch /foo/bar
$ ls -Z foo/
$ podman run --rm -v ./foo:/foo ubi8 touch /foo/bar
```

Original
```
$ podman run --rm --security-opt label=disable ubi8 cat /proc/self/attr/current
$ podman run --rm -v $HOME/.ssh:/ssh --security-opt label=disable ubi8 ls /ssh
```

## 10.9 seccompによるシステムコールの分離
(このコンテンツはChapter 10.9に該当します)

Original
```
$ sed '/mkdir/d' /usr/share/containers/seccomp.json > /tmp/seccomp.json
$ diff /usr/share/containers/seccomp.json /tmp/seccomp.json
$ podman run --rm --security-opt seccomp=/tmp/seccomp.json ubi8 mkdir /foo
$ podman run --rm ubi8 mkdir /foo
```
