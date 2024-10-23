# Настройка SSH на HQ-SRV и BR-SRV

Делаем на всякий случай бэкап конфига:

```
cd /etc/openssh
cp -a sshd_config sshd_config.bak
```

Редактируем файл конфигурации SSH сервера

```
nano sshd_config
```

Изменяем следующие параметры. Не забываем их раскоментировать. Если какой-то параметр не находиться, то просто добавьте его сами

```
Port 2024
MaxAuthTries 3
Banner /etc/openssh/banner
AllowUsers sshuser
```

Создаем файл с баннером

```
nano /etc/openssh/banner
```

Вставляем в него следующие содержимое

```
******************************************************
*                     WARNING!                       *
*                                                    *
*           Access only authorized users!            *
*                                                    *
*  If you not authorized user, close this session!   *
*                                                    *
******************************************************
```

Перезагружаем `SSH`

```
systemctl restart sshd
```

## Проверка

<img src="01.png" width='600'>