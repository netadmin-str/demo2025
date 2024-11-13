# Собираем топология на Esxi

1. Сначала нужно развернуть все виртуальные машины, которые указаны в таблице.

| Машина | RAM,ГБ | CPU | HDD/SDD, ГБ | Шаблон |
| ------ | ------ | --- | ----------- | ------ |
| ISP | 4 | 1 | 10 | «vESR» |
| HQ-RTR | 4 | 1 | 10 | «EcoRouter» |
| BR-RTR | 4 | 1 | 10 | «vESR» |
| HQ-SRV | 2 | 1 | 10 | «ALT-server» |
| BR-SRV | 2 | 1 | 10 | «ALT-server» |
| HQ-CLI | 3 | 2 | 15 | «ALT-workstatison» |

### EcoRouter
Установка `EcoRouter`[->](./ecorouter_install/README.md)

### vESR
Установка `vESR`[->](./install_vesr/README.md)


2. Создаем виртуальные сети.[->](./create_virtual_switch/README.md)

3. Делаем внеполосное управление.[->](./mgmt/README.md)
4. Настройка консоли `vESR`.[->](./settings_vesr_console/README.md)
