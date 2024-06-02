# Описание
Shell скрипт и роль для Ansible.
Автоматизируют настройку роутера на OpenWrt для роутинга по доменам и спискам IP-адресов.
Поможет, если у вас есть отдельный сервер со внутренними доменами, к которым нужно получить доступ,

либо 

Списки берутся с [antifilter.download](https://antifilter.download/)

Использованные ресурсы:
1. https://habr.com/ru/articles/767464/
1. https://4pda.ru/forum/index.php?s=&showtopic=883101&view=findpost&p=93198674

# Использование

Для работы необходимо настроенное на роутере подключение к VPN-серверу

Скачать playbook и темплейты в

```
git clone https://github.com/maksimkurb/ansible-keenetic-domain-routing
cd ansible-keenetic-domain-routing/
```

Добавить роутер в файл hosts в группу keenetic
```
[keenetic]
192.168.1.1 ansible_user=root ansible_password=keenetic
```

Подставить переменные в **group_vars/keenetic.yml**
```
vpn_interface_name: nwg0 # имя интерфейса VPN (можно посмотреть с помощью ifconfig)
lists_dir: "{{ prefix }}/tmp/lst" # Папка, в которую будут сохраняться скачанные списки маршрутов
custom_unblock_hosts: "{{ prefix }}/etc/unblock.txt" # Файл с собственными адресами, которые будут идти через VPN
custom_direct_hosts: "{{ prefix }}/etc/direct.txt" # [NEW!] Файл с собственными адресами, которые всегда будут идти напрямую

ipset_name: rublock # Название списка ip сетей в netfilter (для опытных пользователей)
prefix: /opt # Префикс корневой системы OPKG (для опытных пользователей)
shell: /opt/bin/sh # Оболочка в которой будут запускаться скрипты (для опытных пользователей)
```

Запуск playbook
```
ansible-playbook keenetik_vpn_routing.yml -i hosts
```

После выполнения playbook перезагрузите роутер, после чего он начнёт пробрасывать выбранные домены и подсети через VPN сервер.

### Свои правила маршрутизации
Дополительные адреса можно прописать в следующие файлы:

| Приоритет | Файл | Описание |
| --------- | ---- | -------- |
| 993 | `/opt/etc/routing-direct.txt` | Список адресов, которые **НИКОГДА** не будут идти через VPN, _даже если они есть в файле `routing-vpn.txt` или списках antifilter_ |
| 1000 | `/opt/etc/routing-vpn.txt` | Список адресов, которые будут перенаправлены через VPN, _даже если их нет в списках antifilter_ |

Приоритет: 0 - самый высокий приоритет, 32767 - самый низкий приоритет

## Протестировано на
1. Keenetic Viva KN-1910
1. Zyxel Keenetic Giga II
