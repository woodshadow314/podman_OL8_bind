# Запуск BIND9:9.16 из контейнера ISC
## Подготовительные работы

```bash 
# установка дополнительного ПО
$sudo yum -y install git telnet bash-completion bind-utils mc tree htop vim

# установка Podman & Co. 
$sudo dnf module install container-tools:ol8

# установка только Podman
$sudo yum -y install podman

#granted permission bind ports < 1024 for rootless users
#save configuration permanently, user root only (sudo not work)
echo 'net.ipv4.ip_unprivileged_port_start=0' > /etc/sysctl.d/50-unprivileged-ports.conf
#apply conf
$sudo sysctl --system
```
## Настройки безопасности
```bash 
# установка sealert
$sudo yum -y install setroubleshoot-server
# просмотр предупреждений selinux
$sudo audit2why < /var/log/audit/audit.log
#Вывод анализа причины блокировки SELinux-ом доступа к ресурсу
$sudo sealert -a /var/log/audit/audit.log

# временная смена selinux контекста каталога
chcon -R -t container_file_t -u unconfined_u ~/BIND9
# возврат контекста к определенному политиками
restorecon -R -F -v BIND9/

# поиск политики назначения контекста SELinux заданного для каталога, где Podman хранит тома контейнеров - это "container_file_t"
$sudo semanage fcontext -l | grep .local/share/containers/storage/volumes
/home/[^/]+/\.local/share/containers/storage/volumes/[^/]*/.* all files          unconfined_u:object_r:container_file_t:s0
# создание политики задания контекста SELinux
$sudo semanage fcontext -a -s unconfined_u -t container_file_t '/home/adminos/BIND9/etc/bind(/.*)?'
$sudo semanage fcontext -a -s unconfined_u -t container_file_t '/home/adminos/BIND9/var(/.*)?'
# проверка успешности создания политики
$sudo semanage fcontext -l | grep container_file_t
# применение контекста из политики к  ~/BIND9/
restorecon -R -F -v ~/BIND9/

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
$mkdir ~/BIND9
$git clone https://github.com/woodshadow314/podman_OL8_bind ~/BIND9/
$tree ~/BIND9/


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
        --name=bind_9.16 \
        --restart=always \
        --publish 53:53/udp \
        --publish 53:53/tcp \
        --volume ~/BIND9/etc/bind:/etc/bind \
        --volume ~/BIND9/var/cache/bind:/var/cache/bind \
        --volume /var/lib/bind \
        --volume ~/BIND9/var/log:/var/log \
        docker.io/internetsystemsconsortium/bind9:9.16



```
## Тестирование состояния BIND9
```bash
#temporary installation of additional software for configuration and testing
$podman exec -it bind_9.16 /usr/bin/apt install dnsutils

#проверка доступности внешних DNS серверов
podman exec -it bind_9.16 nslookup ya.ru

#Вывод содержимого примененной конфигурации
#$sudo /usr/sbin/named-checkconf -p -t /chroot/bind/
podman exec -it bind_9.16 /usr/sbin/named-checkconf -p

#вывод загруженных зон - краткий формат
#$sudo /usr/sbin/named-checkconf -z -t /chroot/bind/
podman exec -it bind_9.16 /usr/sbin/named-checkconf -z
```