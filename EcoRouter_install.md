# Установка EcoRouter на Esxi 

Имеем образ EcoRouter в qcow2 формате.

### Шаг 1. Конвертируем qcow2 в vmdk

Используем qemu-img скачать можно [здесь](https://cloudbase.it/downloads/qemu-img-win-x64-2_3_0.zip)

`ipv6.qcow2` - имя файла образа EcoRouter в формате qcow2

`ipv6.vmdk` - имя файла результата конвертации образа EcoRouter (может быть любым) 

```
qemu-img.exe convert -f qcow2 ipv6.qcow2 -O vmdk ipv6.vmdk
```

### Шаг 2. Конвертируем vmdk для Esxi.

Напрямую в `Esxi` прикрутить этот диск к виртуалке так и не получилось.

Поэтому идем обходным путем через `VMware Workstation`

<p align="center">
  <img src="./01.png">
  <img src="./02.png">
</p>

<p align="center">
  <img src="./03.png">
  <img src="./04.png">
</p>

<p align="center">
  <img src="./05.png">
  <img src="./06.png">
</p>


<p align="center">
  <img src="./07.png">
  <img src="./08.png">
</p>

<p align="center">
  <img src="./09.png">
  <img src="./10.png">
</p>

<p align="center">
  <img src="./11.png">
  <img src="./12.png">
</p>

<p align="center">
  <img src="./13.png">
</p>

Запускаем. Будет ругаться на `serial` интерфейс. Ничего страшного потом исправим.

<p align="center">
  <img src="./14.png">
</p>

Выключаем и делаем экспорт в `OVF` формат

<p align="center">
  <img src="./15.png">
</p>

### Шаг 3. Импорт в Esxi.
Переходим в `Esxi` и делаем импорт

<p align="center">
  <img src="./16.png">
  <img src="./17.png">
  <img src="./18.png">
</p>

После импорта виртуалка автоматом запускается. Выключаем ее.

Добавляем `Serial Port`. Через него в дальнейшем можно внеполосно настраивать `EcoRouter`.

<p align="center">
  <img src="./19.png">
</p>

`EcoRouter` пишет все в `Serial Port`. Поэтому в консоли `Esxi` ничего нет.

> [!WARNING]
> Если в виртуалке оставить один сетевой интерфейс, то `EcoRouter` будет задумываться на этапе инициализации `mgmt0` интерфейса и уходить в перезагрузку

```
A start job is running for /sys/sub…devices/mgmt0 (1min 9s / 1min 30s)
```

<p align="center">
  <img src="./20.png">
</p>
<p align="center">
  <img src="./21.png">
</p>

Добавляем несколько сетевых интерфейсов тип `E1000e`. `VMXNET3` он тоже видит, но они всегда в статусе `DOWN`.

<p align="center">
  <img src="./22.png">
</p>

Если в `VM Network` работает `DHCP`, то роутер загрузиться быстрее. Не будет ждать `mgmt0` интерфейса. Я это не проверял, просто предполагаю. 

Если все прошло успешно, то на консоли `Esxi` и `Serial` вы увидите приглашение для ввода логина.

<p align="center">
  <img src="./23.png">
</p>

Проверьте, что порты в `UP`

<p align="center">
  <img src="./24.png">
</p>

