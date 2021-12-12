# Запуск BIND9 из контейнера ISC
## Подготовительные работы

```bash 
# установка дополнительного ПО
$sudo yum -y install git

# установка sealert
$sudo yum -y install setroubleshoot-server

# установка Podman & Co. 
$sudo yum -y install podman

#save configuration permanently
# echo 'net.ipv4.ip_unprivileged_port_start=0' > /etc/sysctl.d/50-unprivileged-ports.conf
#apply conf
$sudo sysctl --system
```

## Подключение конфигурации
```bash
```

## Запуск контейнера

```bash
# загрузка контейнера bind9:9.16 из репозитория
$podman pull docker.io/internetsystemsconsortium/bind9:9.16

$podman run \
        --name=bind9 \
        --restart=always \
        --publish 53:53/udp \
        --publish 53:53/tcp \
        --publish 127.0.0.1:953:953/tcp \
        docker.io/internetsystemsconsortium/bind9:9.16
```