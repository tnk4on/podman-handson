# Chapter 8 Kubernetesとの連携
- レベル:初級
- 見込み時間:15分
- コンテンツ

## 8.2 PodmanでKubernetes YAMLファイルを生成する
(このコンテンツはChapter 8.2に該当します)

Original
```
$ podman rm -f --ignore myapp
$ podman create -p 8080:8080 --name myapp quay.io/rhatdan/myimage
```

Original
```
$ podman kube generate myapp > myapp.yaml
```

Original
```
$ podman image inspect quay.io/rhatdan/myimage | jq .[].User
```

Original
```
$ podman kube generate --type deployment --replicas 2 myapp
```

## 8.3 Kubernetes YAMLからPodmanのPodとコンテナを作成する
(このコンテンツはChapter 8.3に該当します)

Original
```
$ podman rm -f --ignore myapp
$ podman kube play myapp.yaml
```


### Kubernetes YAMLファイルに基づいてPodとコンテナをシャットダウンする
(このコンテンツはChapter 8.3.1に該当します)

```
$ podman kube down myapp.yaml
```

```
$ podman pod ps
```

```
$ podman kube play myapp.yaml
```

### PodmanとKuberentes YAMLファイルを使用してイメージをビルドする
(このコンテンツはChapter 8.3.2に該当します)

Original
```
$ cat > ./Containerfile << _EOF
FROM ubi8-init
RUN dnf -y install httpd; dnf -y clean all
RUN systemctl enable httpd.service
_EOF
$ podman pod rm --all --force
$ podman rm --all --force
$ podman build -t mysystemd .
```

Original
```
$ podman create --rm -p 8080:80 --name myapp -v ./html:/var/www/html:Z mysystemd
$ podman kube generate myapp > myapp2.yaml
$ cat myapp2.yaml
```

Original
```
$ podman pod rm --all --force
$ podman rm --all --force
$ podman rmi mysystemd
```

Original
```
$ mkdir mysystemd
$ mv Containerfile mysystemd/
$ podman kube play --build myapp2.yaml
```

## 8.4 コンテナ内でPodmanを動かす
### Podmanコンテナ内でPodmanを実行する
(このコンテンツはChapter 8.4.1に該当します)

Original
```
$ podman run --privileged quay.io/podman/stable podman version
```

Original
```
$ podman run --user podman quay.io/podman/stable podman version
$ podman run --cap-drop=all --cap-add CAP_SETUID,CAP_SETGID --user podman quay.io/podman/stable podman version
```

