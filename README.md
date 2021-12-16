# Запуск BIND9:9.16 из контейнера ISC
## Подготовительные работы

```bash 
# установка дополнительного ПО
$sudo yum -y install git telnet bash-completion bind-utils mc

# установка sealert
$sudo yum -y install setroubleshoot-server

# установка Podman & Co. 
$sudo dnf module install container-tools:ol8

# установка только Podman
$sudo yum -y install podman

#granted permission bind ports < 1024 for rootless users
#save configuration permanently, user root only (sudo not work)
echo 'net.ipv4.ip_unprivileged_port_start=0' > /etc/sysctl.d/50-unprivileged-ports.conf
#apply conf
$sudo sysctl --system

#Firewalld settings
$sudo firewall-cmd --add-service=dns  --permanent
$sudo firewall-cmd --remove-service=dhcpv6-client --permanent
$sudo firewall-cmd --reload
$sudo firewall-cmd --list-all
```

## Подключение конфигурации
```bash
```

## Запуск и настройка контейнера

```bash
# загрузка контейнера bind9:9.16 из репозитория
$podman pull docker.io/internetsystemsconsortium/bind9:9.16

$podman run \
        --name=bind9.16 \
        --restart=always \
        --publish 53:53/udp \
        --publish 53:53/tcp \
        --publish 127.0.0.1:953:953/tcp \
        --volume /etc/bind \
        --volume /var/cache/bind \
        --volume /var/lib/bind \
        --volume /var/log \
        docker.io/internetsystemsconsortium/bind9:9.16

$podman run \
        --name=bind9.16 \
        --restart=always \
        --publish 53:53/udp \
        --publish 53:53/tcp \
        --volume ~/BIND9/etc/bind/named.conf:/etc/bind/named.conf \
        --volume /var/cache/bind \
        --volume /var/lib/bind \
        --volume /var/log \
        docker.io/internetsystemsconsortium/bind9:9.16



```
## Тестирование состояния BIND9
```bash
#temporary installation of additional software for configuration and testing
$podman exec -it bind9.16 /usr/bin/apt install dnsutils

#проверка доступности внешних DNS серверов
podman exec -it bind9.16 nslookup ya.ru

#Вывод содержимого примененной конфигурации
#$sudo /usr/sbin/named-checkconf -p -t /chroot/bind/
/usr/sbin/named-checkconf -p

#вывод загруженных зон - краткий формат
#$sudo /usr/sbin/named-checkconf -z -t /chroot/bind/
/usr/sbin/named-checkconf -z
```