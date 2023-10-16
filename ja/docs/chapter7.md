# Chapter 7 systemdとの統合
- レベル:初級
- 見込み時間:15分
- コンテンツ


## 7.1 コンテナ内でsystemdを実行する
(このコンテンツは本書 7.1に該当します)

Original
```
$ podman pull ubi8-init
$ podman inspect ubi8-init --format '{{ .Config.Cmd }}'
```


## systemdモードのPodmanコンテナ
(このコンテンツは本書 7.1.2に該当します)

Original
```
$ podman create --rm --name SystemD -ti --systemd=always ubi8-init sh
$ podman inspect SystemD --format '{{ .Config.StopSignal}}'
```

Original
```
$ podman start --attach SystemD
sh-4.4# mount | grep -e /tmp -e /run | head -2
sh-4.4# printenv container
```

Original
```
$ podman run -ti ubi8-init
```


### systemdコンテナ内でのApacheサービスの実行
(このコンテンツは本書 7.1.3に該当します)

Original
```
$ mkdir /tmp/pia-systemd-httpd
$ cat << _EOF > /tmp/pia-systemd-httpd/Containerfile
FROM ubi8-init
RUN dnf -y install httpd; dnf -y clean all
RUN systemctl enable httpd.service
_EOF
$ podman build -t my-systemd /tmp/pia-systemd-httpd/
```

Original
```
$ podman run -d --rm -p 8080:80 -v ./html:/var/www/html:Z my-systemd
$ podman ps
$ podman logs 7675617e5b8b
```


## 7.2 journaldによるログとイベントの管理
### ログの管理
(このコンテンツは本書 7.2.1に該当します)

Original
```
$ podman info --format '{{ .Host.LogDriver }}'
$ mkdir -p $HOME/.config/containers/containers.conf.d
$ cat > $HOME/.config/containers/containers.conf.d/log_driver.conf << _EOF
[containers]
log_driver="journald"
_EOF
$ podman info --format '{{ .Host.LogDriver }}'
```


### イベントの管理
(このコンテンツは本書 7.2.2に該当します)

Original
```
$ podman events --filter event=start --since 1h
$ podman info --format '{{ .Host.EventLogger }}'
```


## 7.3 起動時におけるコンテナの自動起動
### systemdサービスとしてのPodmanコンテナ
(このコンテンツは本書 7.3.2に該当します)

Original
```
$ podman create -p 8080:8080 --name myapp quay.io/rhatdan/myimage
```

Original
```
$ cat $HOME/.config/systemd/user/myapp.service
```

Original
```
$ systemctl --user daemon-reload
$ systemctl --user start myapp
$ systemctl --user status myapp
```

Original
```
$ systemctl --user stop myapp
```


### Podmanコンテナの管理に使用するsystemdユニットファイルの配布
(このコンテンツは本書 7.3.3に該当します)

Original
```
$ podman generate systemd --new myapp > $HOME/.config/systemd/user/myapp-new.service
$ cat $HOME/.config/systemd/user/myapp-new.service
```


### Podmanコンテナの自動更新
(このコンテンツは本書 7.3.4に該当します)

Original
```
$ systemctl --user stop myapp
$ podman rm myapp --force -t 0
```

Original
```
$ podman create --label "io.containers.autoupdate=registry" -p 8080:8080 --name myapp quay.io/rhatdan/myimage
$ podman generate systemd myapp --new > $HOME/.config/systemd/user/myapp-new.service
```

Original
```
$ systemctl --user daemon-reload
$ systemctl --user start myapp-new
```

Original
```
$ podman exec -i myapp bash -c 'cat > /var/www/html/index.html' << _EOF
<html>
<head>
</head>
<body>
<h1>Welcome to the new Hello World<h1>
</body>
</html>
_EOF
```

Original
```
$ podman commit myapp quay.io/rhatdan/myimage-new
$ podman push quay.io/rhatdan/myimage-new quay.io/rhatdan/myimage
$ podman rmi quay.io/rhatdan/myimage-new
```

Original
```
$ podman auto-update
```


## 7.6 Podmanコンテナとソケットアクティベーション
(このコンテンツは本書 7.6に該当します)

Original
```
$ systemctl --user stop myapp.service
$ cat > $HOME/.config/systemd/user/myapp.socket <<_EOF
[Unit]
Description=myapp socket service
PartOf=myapp.service
[Socket]
ListenStream=127.0.0.1:8080
[Install]
WantedBy=sockets.target
_EOF
$ systemctl --user enable --now myapp.socket
$ podman ps
```
