# Установка EcoRouter на Esxi 

Имеем образ EcoRouter в qcow2 формате.

Шаг 1. Конвертируем qcow2 в vmdk

Используем qumu-img скачать можно [здесь](https://cloudbase.it/downloads/qemu-img-win-x64-2_3_0.zip)

ipv6.qcow2 - имя файла образа EcoRouter в формате qcow2

ipv6.vmdk - имя файла результата конвертации образа EcoRouter (может быть любым) 

```
qemu-img.exe convert -f qcow2 ipv6.qcow2 -O vmdk ipv6.vmdk
```

Напрямую в Esxi прикрутить этот диск к виртуалке так и не получилось.

Поэтому идем обходным путем через VMware Workstation

<p align="center">
  <img src="./01.png">
</p>

<p align="center">
  <img src="./02.png">
</p>

<p align="center">
  <img src="./03.png">
</p>

<p align="center">
  <img src="./04.png">
</p>

<p align="center">
  <img src="./05.png">
</p>

<p align="center">
  <img src="./06.png">
</p>

<p align="center">
  <img src="./07.png">
</p>

<p align="center">
  <img src="./08.png">
</p>

<p align="center">
  <img src="./09.png">
</p>

<p align="center">
  <img src="./10.png">
</p>

<p align="center">
  <img src="./11.png">
</p>

<p align="center">
  <img src="./12.png">
</p>

<p align="center">
  <img src="./13.png">
</p>

Запускаем. Будет ругаться на serial интерфейс. Ничего страшного потом исправим

<p align="center">
  <img src="./14.png">
</p>

Выключаем и делаем экспорт в OVF формат

<p align="center">
  <img src="./15.png">
</p>

Переходим в Esxi и делаем импорт

<p align="center">
  <img src="./16.png">
</p>

<p align="center">
  <img src="./17.png">
</p>

<p align="center">
  <img src="./18.png">
</p>

После импорта виртуалка автоматом запускается. Выключаем ее.