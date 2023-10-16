# Chapter 9 サービスとしてのPodman
- レベル:初級
- 見込み時間:15分
- コンテンツ

## 9.1 Podmanサービスの紹介
(このコンテンツは本書 9.1に該当します)

Original
```
$ podman system service
```


### systemdサービス
(このコンテンツは本書 9.1.1に該当します)

Original
```
$ systemctl --user enable podman.socket
$ systemctl --user start podman.socket
$ ls $XDG_RUNTIME_DIR/podman/podman.sock
```

Original
```
$ curl -s --unix-socket $XDG_RUNTIME_DIR/podman/podman.sock http://d/v1.0.0/libpod/version | jq
```


## 9.2 PodmanがサポートするAPI
(このコンテンツは本書 9.2に該当します)

Original
```
$ curl -s --unix-socket $XDG_RUNTIME_DIR/podman/podman.sock http://d/v1.0.0/libpod/images/json | jq
```

Original
```
$ podman pod create --name mypod
$ curl -s --unix-socket $XDG_RUNTIME_DIR/podman/podman.sock http://d/v1.0.0/libpod/pods/json | jq
$ curl -s --unix-socket $XDG_RUNTIME_DIR/podman/podman.sock http://d/v1.0.0/pods/json
```

## 9.3 Podmanとやり取りするためのPythonライブラリ
### Podman APIでのdocker-pyの使用
(このコンテンツは本書 9.3.1に該当します)

Original
```
$ sudo dnf install -y python-docker
```

Original
```
$ cat > images.py << _EOF
import docker
client=docker.DockerClient(base_url='unix:/run/user/1000/podman/podman.sock')
print(client.images.list(all=True))
_EOF
$ python images.py
```


### podman-pyとPodman APIの使用
(このコンテンツは本書 9.3.2に該当します)

Original
```
$ sudo dnf install -y python-podman
$ cat > podman-images.py << _EOF
import podman
client=podman.PodmanClient()
print(client.images.list())
_EOF
$ python podman-images.py
```

Original
```
$ cat >> podman-images.py << _EOF
for i in client.pods.list():
  print(i.attrs)
_EOF
$ python podman-images.py
```

## 9.4 Podmanサービスでdocker-composeを使用する
(このコンテンツは本書 9.4に該当します)

Original
```
$ sudo dnf -y install docker-compose
$ systemctl --user start podman.socket
$ curl -H "Content-Type: application/json" --unix-socket $XDG_RUNTIME_DIR/podman/podman.sock http://localhost/_ping
OK
```

Original
```
$ export DOCKER_HOST=unix://$XDG_RUNTIME_DIR/podman/podman.sock
$ mkdir example
$ mv ./html example
$ cd example
$ cat > docker-compose.yaml << _EOF
version: "3.7"
services:
  myapp:
    image: quay.io/rhatdan/myimage:latest
    volumes:
      - ./html:/var/www/html
      - myapp_vol:/vol
    ports:
      - 8080:80
volumes:
  myapp_vol: {}
_EOF
```

Original
```
$ podman pod rm --all --force
$ podman rm --all --force
$ podman rmi --all --force
$ podman volume rm --all --force
```

Original
```
$ docker-compose up
```

Original
```
$ podman ps --format "{{.ID}} {{.Image}} {{.Ports}} {{.Names}}"
$ podman volume ls
```

Original
```
^C
$ podman ps --format "{{.ID}} {{.Image}} {{.Ports}} {{.Names}}"
$ podman ps -a --format "{{.ID}} {{.Image}} {{.Ports}} {{.Names}}"
$ docker-compose down
$ podman ps -a --format "{{.ID}} {{.Image}} {{.Ports}} {{.Names}}"
```

## 9.5 podman --remote
(このコンテンツは本書 9.5に該当します)

Original
```
$ podman --remote version
$ podman --remote run ubi8 echo hi
```



### リモート接続
(このコンテンツは本書 9.5.2に該当します)

Original
```
$ sudo systemctl enable --now sshd
$ systemctl --user enable --now podman.socket
$ sudo loginctl enable-linger $USER
$ podman --remote info
```


### クライアントマシンにSSHを設定する
(このコンテンツは本書 9.5.3に該当します)

Original
```
$ ssh-keygen -t ed25519
$ ssh-copy-id myuser@192.168.122.
```


### 接続を設定する
(このコンテンツは本書 9.5.4に該当します)

Original
```
$ podman system connection add server1 --identity ~/.ssh/id_ed25519 ssh://myuser@192.168.122.1/run/user/1000/podman/podman.sock
$ podman system connection list
$ podman --remote info
```