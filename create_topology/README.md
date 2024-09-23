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


2. Создаем виртуальные сети.[->](./create_virtual_switch/README.md)

3. Делаем внеполосное управление.[->](./mgmt/README.md)
