# Описание
Playbook для Ansible, автоматизирующий настройку обхода блокировок РКН через VPN на роутере Keenetic с модулем OPKG

Списки берутся с [antifilter.download](https://antifilter.download/)

Использованные ресурсы:
1. https://itdog.info/tochechnyj-obhod-blokirovok-rkn-na-routere-s-openwrt-s-pomoshhyu-wireguard-i-dnscrypt/ или https://habr.com/ru/post/440030/
1. https://habr.com/ru/post/428992/
1. https://4pda.ru/forum/index.php?s=&showtopic=883101&view=findpost&p=93198674

# Использование

Для работы необходим VPN сервер вне зоны действия РКН и настроенное подключение к нему

Скачать playbook и темплейты в

```
git clone https://github.com/maksimkurb/ansible-keenetic-hirkn
cd ansible-keenetic-hirkn/
```

Добавить роутер в файл hosts в группу keenetic
```
[keenetic]
192.168.1.1 ansible_user=root ansible_password=keenetic
```

Подставить переменные в **group_vars/keenetic.yml**
```
vpn_interface_name: nwg0 # имя интерфейса VPN (можно посмотреть с помощью ifconfig)
lists_dir: "{{ prefix }}/tmp/lst" # Папка, в которую будут сохраняться списки заблокированных маршрутов
custom_unblock_hosts: "{{ prefix }}/etc/unblock.txt" # Файл с собственными адресами, которые будут идти через VPN
custom_direct_hosts: "{{ prefix }}/etc/direct.txt" # [NEW!] Файл с собственными адресами, которые всегда будут идти напрямую

ipset_name: rublock # Название списка ip сетей в netfilter (для опытных пользователей)
prefix: /opt # Префикс корневой системы OPKG (для опытных пользователей)
shell: /opt/bin/sh # Оболочка в которой будут запускаться скрипты (для опытных пользователей)
```

Запуск playbook
```
ansible-playbook hirkn.yml -i hosts
```

После выполнения playbook перезагрузите роутер, после чего он начнёт выполнять обход блокировок через VPN сервер.

### Свои правила маршрутизации
Дополнительные адреса можно прописать в следующие файлы:

| Приоритет | Файл | Описание |
| --------- | ---- | -------- |
| 993 | `/opt/etc/direct.txt` | Список адресов, которые **НИКОГДА** не будут идти через VPN, _даже если они заблокированы РКН или есть в файле `unblock.txt`_ |
| 1000 | `/opt/etc/unblock.txt` | Список адресов, которые будут идти через VPN, _даже если они не заблокированы РКН_ |

### Дополнительные правила
Можно указать дополнительные IP и адреса, которые ВСЕГДА будут идти через VPN, вписав их в файл `/opt/etc/unblock.txt`

## Протестировано на
1. Keenetic Viva KN-1910
1. Zyxel Keenetic Giga II