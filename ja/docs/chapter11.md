# Chapter 11 その他のセキュリティに関する考慮事項
- レベル:初級
- 見込み時間:15分
- コンテンツ

## 11.1 デーモンとfork/execモデルとの比較
### docker.sockへのアクセス
(このコンテンツは本書 11.1.1に該当します)

Original
```
# ls -l /run/docker.sock
```

Original
```
$ docker run registry.access.redhat.com/ubi8-micro echo hi
```

Original
```
$ docker run -ti --name hack -v /:/host --privileged registry.access.redhat.com/ubi8-micro chroot /host
# cat /etc/shadow
$ docker rm hack
```


### 監査とロギング
(このコンテンツは本書 11.1.2に該当します)

Original
```
$ cat /proc/self/loginuid
$ sudo cat /proc/self/loginuid
```

Original
```
$ podman run -d ubi8-micro sleep 20
$ podman inspect -l --format '{{ .State.Pid }}'
$ cat /proc/119394/loginuid
```

Original
```
$ docker run -d registry.access.redhat.com/ubi8-micro sleep 20
$ docker inspect df2302cf8c6 --format '{{ .State.Pid }}'
$ cat /proc/120022/loginuid
```

Original
```
# auditctl -w /etc/passwd -p wa -k passwd
# docker run --privileged -v /:/host registry.access.redhat.com/ubi8-micro:latest touch /host/etc/passwd
```

Original
```
# podman run --privileged -v /:/host registry.access.redhat.com/ubi8-micro:latest touch /host/etc/passwd
# ausearch -k passwd -i
```

## 11.2 Podmanによる機密情報の取り扱い
(このコンテンツは本書 11.2に該当します)

Original
```
$ echo "This is my secret" > /tmp/secret
$ podman secret create my_secret /tmp/secret
$ podman run --rm --secret my_secret ubi8 cat /run/secrets/my_secret
$ podman run --secret my_secret,type=env --name secret_ctr ubi8 bash -c 'echo $my_secret'
```

Original
```
$ podman commit secret_ctr secret_img
$ podman image inspect secret_img --format '{{ .Config.Env }}'
```

## 11.3 Podmanによるイメージの信頼
(このコンテンツは本書 11.3に該当します)

Original
```
$ sudo cp /etc/containers/policy.json /tmp
$ sudo podman image trust set -t reject docker.io
$ podman pull alpine
$ sudo podman image trust set -t accept docker.io/library
$ podman pull alpine
$ podman pull bitnami/nginx
```

Original
```
$ cat /etc/containers/policy.json
```

Original
```
$ podman image trust show
```

Original
```
$ sudo podman image trust set --type=reject default
$ podman image trust show
$ sudo cp /tmp/policy.json /etc/containers/policy.json
```

### Podmanによるイメージの署名
(このコンテンツは本書 11.3.1に該当します)

Original
```
$ gpg --batch --passphrase '' --quick-gen-key dwalsh@redhat.com default default
$ sudo cp /etc/containers/registries.d/default.yaml /etc/containers/policy.json /tmp
```

Original
```
$ sudo podman pull quay.io/rhatdan/myimage
$ podman login quay.io/rhatdan
$ sudo -E GNUPGHOME=$HOME/.gnupg \
podman push --tls-verify=false --sign-by dwalsh@redhat.com quay.io/rhatdan/myimage
$ sudo ls /var/lib/containers/sigstore/rhatdan/
```

Original
```
$ echo " sigstore: http://localhost:8000" | sudo tee --append /etc/containers/registries.d/default.yaml
$ cd /var/lib/containers/sigstore && python3 -m http.server

$ podman rmi quay.io/rhatdan/myimage
$ sudo podman image trust set -f /tmp/publickey.gpg quay.io/rhatdan
$ gpg --output /tmp/publickey.gpg --armor --export dwalsh@redhat.com
$ podman pull quay.io/rhatdan/myimage
$ podman pull quay.io/rhatdan/podman
$ sudo cp /tmp/default.yaml /etc/containers/registries.d/default.yaml
$ sudo cp /tmp/policy.json /etc/containers/policy.json
```

## 11.4 Podmanによるイメージスキャン
(このコンテンツは本書 11.4に該当します)

Original
```
$ podman image mount ubi8
$ podman unshare
# podman image mount
# mnt=$(podman image mount ubi8)
# echo $mnt
# cd $mnt
# /usr/bin/find . -user root -perm -4000
```

### 読み取り専用コンテナ
(このコンテンツは本書 11.4.1に該当します)

Original
```
$ podman run --read-only ubi8 touch /foo
$ podman run --read-only ubi8 touch /run/foo
$ podman run --read-only-tmpfs=false --read-only ubi8 touch /run/foo
```