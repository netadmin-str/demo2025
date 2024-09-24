# Распределение IP адресов

| Имя устройства | Интерфейс | IP         | Маска   | Шлюз   |
| -------------- | --------- | ---------- | ------- | ------ |
| ISP            | gi1/0/1   | DHCP       |         |        |
|                | gi1/0/2   | DHCP  |  | 
|                | gi1/0/3   | DHCP |  | 
| HQ-RTR | 4 | 1 | 10 | «EcoRouter» 
| BR-RTR | 4 | 1 | 10 | «vESR» |
| HQ-SRV | 2 | 1 | 10 | «ALT-server» |
| BR-SRV | 2 | 1 | 10 | «ALT-server» |
| HQ-CLI | 3 | 2 | 15 | «ALT-workstatison» |