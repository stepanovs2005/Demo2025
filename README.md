# Demo2025

**Решение Demo2025 по специальности 09.02.06 «Сетевое и системное администрирование»**

Данный репозиторий содержит пошаговую инструкцию по настройке сетевой инфраструктуры в соответствии с предложённой топологией. В документации описаны этапы базовой настройки, включая конфигурацию имён устройств, настройку IPv4, NAT, VLAN, туннелей GRE с динамической маршрутизацией (OSPF), а также настройку DNS и DHCP.

> **Важно:**  
> Все примеры с указанием интерфейсов, IP-адресов, масок и шлюзов приведены в качестве примера. В вашей сети данные параметры могут отличаться. Перед внесением изменений обязательно сверяйте настройки с актуальной топологией и требованиями вашей инфраструктуры.

---

## Содержание

- [Описание задания](#описание-задания)
- [Диапазоны IP-адресов по RFC1918](#диапазоны-ip-адресов-по-rfc1918)
- [Разделение IP-адресов по VLAN](#разделение-ip-адресов-по-vlan)
- [Адресация и шлюзы для настройки ISP](#адресация-и-шлюзы-для-настройки-isp)
- [Решение](#решение)
  - [1. Настройка имён устройств](#1-настройка-имен-устройств)
  - [2. Конфигурация IPv4 на JeOS](#2-конфигурация-ipv4-на-jeos)
  - [3. Перезагрузка сети и обновление пакетов](#3-перезагрузка-сети-и-обновление-пакетов)
  - [4. Настройка сетевых адаптеров](#4-настройка-сетевых-адаптеров)
  - [5. Настройка NAT (маскарадинга)](#5-настройка-nat-маскарадинга)
  - [6. Включение пересылки пакетов](#6-включение-пересылки-пакетов)
  - [7. Настройка сети для HQ-RTR](#7-настройка-сети-для-hq-rtr)
  - [8. Настройка NAT для офиса HQ](#8-настройка-nat-для-офиса-hq)
  - [9. Настройка сети для BR-RTR](#9-настройка-сети-для-br-rtr)
  - [10. Настройка NAT для офиса BR](#10-настройка-nat-для-офиса-br)
  - [11. Создание локальных учётных записей на HQ-RTR](#11-создание-локальных-учетных-записей-на-hq-rtr)
  - [12. Создание локальных учётных записей на BR-RTR](#12-создание-локальных-учетных-записей-на-br-rtr)
  - [13. Настройка VLAN](#13-настройка-vlan)
  - [14. Настройка HQ-SRV](#14-настройка-hq-srv)
  - [15. Создание локальных учётных записей на HQ-SRV](#15-создание-локальных-учетных-записей-на-hq-srv)
  - [16. Настройка BR-SRV](#16-настройка-br-srv)
  - [17. Настройка безопасного удалённого доступа](#17-настройка-безопасного-удаленного-доступа)
  - [18. Настройка динамической маршрутизации](#18-настройка-динамической-маршрутизации)
  - [19. Настройка DNS и DHCP](#19-настройка-dns-и-dhcp)

---

## Описание задания

<details>
  <summary>Модуль №1: Настройка сетевой инфраструктуры (нажмите для развертывания)</summary>

**Текст задания:**  
Вид аттестации/уровень ДЭ: ПА, ГИА ДЭ БУ, ГИА ДЭ ПУ (инвариантная часть)

**Задание:**  
Необходимо разработать и настроить инфраструктуру информационно-коммуникационной системы согласно предложённой топологии (см. Рисунок 1). Задание включает базовую настройку устройств:
- Присвоение имён устройствам (используйте полные доменные имена);
- Расчёт IP-адресации;
- Настройку коммутации и маршрутизации.

При проектировании и настройке сети следует вести отчёт, включающий таблицы и схемы, предусмотренные в задании. Итоговый отчёт должен содержать одну сводную таблицу и пять отчётов о ходе работы. По завершении работы итоговый отчёт необходимо сохранить на рабочем месте.

**Рисунок 1. Топология сети**

![Топология сети](https://github.com/user-attachments/assets/5e93864e-93db-42b9-ac98-494d5679c599)

**Таблица 1**

| Машина    | RAM (ГБ) | CPU | HDD/SDD (ГБ) | OS                                     |
|-----------|----------|-----|--------------|-----------------------------------------|
| ISP       | 1        | 1   | 10           | ОС Альт JeOS/Linux или аналог           |
| HQ-RTR    | 1        | 1   | 10           | ОС EcoRouter или аналог                 |
| BR-RTR    | 1        | 1   | 10           | ОС EcoRouter или аналог                 |
| HQ-SRV    | 2        | 1   | 10           | ОС Альт Сервер или аналог               |
| BR-SRV    | 2        | 1   | 10           | ОС Альт Сервер или аналог               |
| HQ-CLI    | 3        | 2   | 15           | ОС Альт Рабочая Станция или аналог        |
| **Итого** | **10**   | **7** | **65**     | —                                       |

**Основные этапы работы:**

1. **Базовая настройка устройств:**
   - Задать имена устройств согласно топологии (используйте полные доменные имена).
   - Выполнить конфигурацию IPv4 на всех устройствах, используя адреса из приватного диапазона (RFC1918).
   - Ограничить количество адресов для локальных сетей (VLAN100 – до 64 адресов, VLAN200 – до 16, для BR-SRV – до 32, для VLAN999 – до 8 адресов).
   - Занести сведения об адресации в отчёт (см. пример Таблицы 3).

2. **Настройка ISP:**
   - Настроить адресацию на интерфейсах:
     - Интерфейс, подключённый к магистральному провайдеру, получает адрес по DHCP.
     - Настроить маршруты по умолчанию там, где это необходимо.
     - Для подключения HQ-RTR использовать сеть 172.16.4.0/28, для BR-RTR – 172.16.5.0/28.
     - Настроить динамическую NAT-трансляцию на ISP для доступа к Интернету через HQ-RTR и BR-RTR.

3. **Создание локальных учётных записей:**  
   На серверах HQ-SRV и BR-SRV – создать пользователя **sshuser** с UID 1010, паролем **P@ssw0rd** и правами sudo без повторной аутентификации.  
   На маршрутизаторах HQ-RTR и BR-RTR – создать пользователя **net_admin** с паролем **P@$$word**, имеющего максимальные привилегии.

4. **Настройка виртуального коммутатора на HQ-RTR:**  
   На интерфейсе, подключённом к офису HQ, настроить VLAN:
   - Сервер HQ-SRV – VLAN ID 100.
   - Клиент HQ-CLI – VLAN ID 200.
   - Сеть для управления – VLAN ID 999.  
   Подробности конфигурации VLAN задокументировать в отчёте.

5. **Настройка безопасного удалённого доступа:**  
   На серверах HQ-SRV и BR-SRV настроить SSH:
   - Использовать порт 2024.
   - Разрешить доступ только пользователю **sshuser**.
   - Ограничить число попыток входа до двух.
   - Установить баннер с текстом «Authorized access only».

6. **Настройка IP-туннеля между офисами HQ и BR:**  
   Выбрать технологию GRE или IP-in-IP и задокументировать настройки в отчёте.

7. **Настройка динамической маршрутизации:**  
   Обеспечить доступ к ресурсам между офисами с использованием протокола с состоянием канала (link-state) – выбор конкретного протокола оставляется на усмотрение.  
   Маршрутизаторы обмениваются маршрутами только между собой, с защитой протокола посредством парольной аутентификации.

8. **Настройка динамической NAT-трансляции:**  
   Настроить NAT для обоих офисов для обеспечения доступа к Интернету.

9. **Настройка DHCP:**  
   Для HQ настроить DHCP на HQ-RTR (сервер) и HQ-CLI (клиент). Исключить из выдачи адрес маршрутизатора, указать шлюз и DNS, а также задать DNS-суффикс **au-team.irpo**.

10. **Настройка DNS:**  
    Основной DNS-сервер – HQ-SRV. Он должен обеспечивать обратное разрешение имён (IP ⇄ имя) согласно Таблице 2. В качестве сервера пересылки использовать общедоступный DNS.

11. **Настройка часового пояса:**  
    Установить часовой пояс на всех устройствах в соответствии с местом проведения экзамена.

</details>

---

## Диапазоны IP-адресов по RFC1918

| Диапазон             | CIDR           | Количество адресов | Пример диапазона                  |
|----------------------|----------------|--------------------|-----------------------------------|
| Класс A              | 10.0.0.0/8     | 16 777 216         | 10.0.0.0 – 10.255.255.255           |
| Класс B              | 172.16.0.0/12  | 1 048 576          | 172.16.0.0 – 172.31.255.255          |
| Класс C              | 192.168.0.0/16 | 65 536             | 192.168.0.0 – 192.168.255.255        |

---

## Разделение IP-адресов по VLAN

| VLAN ID | Назначение      | Сеть         | Маска | Количество адресов | Диапазон IP-адресов            |
|---------|-----------------|--------------|-------|--------------------|--------------------------------|
| 100     | Офис HQ-SRV     | 192.168.10.0 | /26   | 64                 | 192.168.10.0 – 192.168.10.63    |
| 200     | Офис HQ-CLI     | 192.168.20.0 | /28   | 16                 | 192.168.20.0 – 192.168.20.15    |
| 300     | Офис BR-SRV     | 192.168.30.0 | /27   | 32                 | 192.168.30.0 – 192.168.30.31    |
| 999     | Сеть управления | 192.168.99.0 | /29   | 8                  | 192.168.99.0 – 192.168.99.7     |

---

## Адресация и шлюзы для настройки ISP

| Подключение         | Сеть           | IP-адрес (устройство)    | IP-адрес (ISP)   | Шлюз по умолчанию (для устройства внутри сети) |
|---------------------|----------------|--------------------------|------------------|-------------------------------------------------|
| HQ-RTR – ISP        | 172.16.4.0/28  | 172.16.4.2 (HQ-RTR)      | 172.16.4.1       | 172.16.4.2                                      |
| BR-RTR – ISP        | 172.16.5.0/28  | 172.16.5.2 (BR-RTR)      | 172.16.5.1       | 172.16.5.2                                      |

## Табличка адресов ip 
| Имя              | IP                               | Шлюз         |
|------------------|-----------------------------------|--------------|
| ISP-HQ           | 172.16.4.2/28                    | 172.16.4.1/28 |
| ISP-BR           | 172.16.5.2/28                    | 172.16.5.1/28 |
| HQ-RTR (VLAN100)  | 192.168.10.1/26                  | 192.168.10.1 | 
| HQ-RTR (VLAN200)  | 192.168.20.1/28                  | 192.168.20.1 |
| HQ-RTR (VLAN999)  | 192.168.99.1/29                  | 192.168.99.1 | 
| HQ-CLI           | 192.168.20.2/28, 192.168.99.2/29  | 192.168.20.1/28|
| HQ-SRV           | 192.168.10.2/26                  | 192.168.10.1/26 |
| BR-RTR           | 192.168.30.1/27                  | 172.16.5.2/28 |
| BR-SRV           | 192.168.30.2/27                  | 192.168.30.1/27 |

https://jodies.de/ipcalc - ip-калькулятор

![image](https://github.com/user-attachments/assets/da181112-1d6d-4da9-bc8c-7bacc63d12d5)

---
Советую сразу на cli который клиент отключить Network-manager
## Решение
### 0.0 Настройка временых зон
Делаем везде одной команой кроме isp cначало надо установить tzdata смотрите ниже
```bash
timedatectl set-timezone Asia/Novosibirsk
```
На ISP нету предустановленных часовых поясов потому их надо установить командой 
```bash
apt-get install tzdata -y
```
После прописываем комаду выше чтобы сделать часовой пояс Новосибирск
### 1. Настройка имён устройств

Присваиваем полные доменные имена устройствам согласно топологии:

```bash
hostnamectl set-hostname isp && exec bash
hostnamectl set-hostname hq-rtr.au-team.irpo && exec bash
hostnamectl set-hostname br-rtr.au-team.irpo && exec bash
hostnamectl set-hostname hq-srv.au-team.irpo && exec bash
hostnamectl set-hostname hq-cli.au-team.irpo && exec bash
hostnamectl set-hostname br-srv.au-team.irpo && exec bash
```

---

### 2. Конфигурация IPv4 на JeOS

#### 2.1 Задание hostname (если не задан)
```bash
hostnamectl set-hostname isp && exec bash
```

#### 2.2 Проверка получения настроек DHCP (isp) 

Проверьте командой:
```bash
ip addr
```
После чего пишем комнаду чтобы обновить спискок пакетов и установить всё необходимое
```bash
apt-get update && apt-get install mc tzdata iptables -y
```
После чего создаём интерфейсы (не забудь поменять сети br с hq в настройках это если что для Стёпы)

настройте DHCP, отредактировав соответствующий конфигурационный файл по пути (`/etc/net/ifaces/ens192/options`) внутри него напишите
```bash
BOOTPROTO=dhcp
TYPE=eth
```
---
Перезапускаем сеть командой 
```bash
systemctl restart network
```
---

### 3. Создание и настройка сетевых адаптеров (НА ISP В СТОРОНУ ПОДСЕТЕЙ HQ И BR)

1. Определите имя сетевого адаптера командой:
   ```bash
   ip addr
   ```
У меня сначала идёт hq потом br у вас будет наоборот если вы не поменяете в настройках виртуальной машины  ISP
2. Создайте каталог командной-для интерфейса для ens224:
   ```bash
   mkdir /etc/net/ifaces/ens224
   ```
3. Отредактируйте файл настроек:
   ```bash
   mcedit /etc/net/ifaces/ens224/options
   ```
   Пример содержимого:
   ```bash
   BOOTPROTO=static
   TYPE=eth
   ```
4. Создайте файл для задания IP-адреса (например, для подсети HQ):
   ```bash
   mcedit /etc/net/ifaces/ens224/ipv4address
   ```
   Пример:
   ```
   172.16.4.1/28
   ```
5. Аналогичным образом задайте IP-адреса для ens256 в сторону HQ в моём случае это BR и перезагрузите сеть командой:
   ```bash
   systemctl restart network
   ip addr - для проверки что сделали правильно
   ```
---

### 4. Настройка NAT (На ISP)

Для обеспечения доступа к Интернету настроем маскарадинг для соответствующих подсетей.

IP АДРЕСА БУДУТ ДРУГИЕ ЭТИ АДРЕСА ВЫБРАНЫ В КАЧЕСТВЕ ПРИМЕРА!!!!!!!!!

#### 4.1 Пример для подсети HQ-RTR:
```bash
iptables -t nat -A POSTROUTING -o ens192 -s 172.16.4.0/28 -j MASQUERADE
```

#### 4.2 Пример для подсети BR-RTR:
```bash
iptables -t nat -A POSTROUTING -o ens192 -s 172.16.5.0/28 -j MASQUERADE
```

Сохраните правила:
```bash
iptables-save > /etc/sysconfig/iptables
systemctl enable --now iptables
```
---

### 5. Включение пересылки пакетов

Отредактируйте файл командой ниже по пути `/etc/net/sysctl.conf`:
```bash
mcedit /etc/net/sysctl.conf
```

   - Измените строку net.ipv4.ip_forward:
     ```diff
     -#net.ipv4.ip_forward = 0
     +net.ipv4.ip_forward = 1
     ```
Примените изменения командой:
```bash
systemctl restart network
```

---

### 6. Настройка сети для HQ-RTR

#### 6.0 Задание Часового пояса (если не задан)
```bash
timedatectl set-timezone Asia/Novosibirsk
```

#### 6.1 Задание hostname (если не задан)
```bash
hostnamectl set-hostname hq-rtr.au-team.irpo && exec bash
```

#### 6.2 Задание IP-адреса

Создайте или отредактируйте файл:
```bash
mcedit /etc/net/ifaces/ens192/ipv4address
```
ЭТО ПРИМЕР У ВАС БУДУТ ДРУГИЕ ПОДСЕТИ
Пишем:
```
172.16.4.2/28
```

#### 6.3 Настройка маршрута по умолчанию

Создайте или отредактируйте файл:
```bash
mcedit /etc/net/ifaces/ens192/ipv4route
```
Пример:
```
default via 172.16.4.1
```

#### 6.4 Настройка DNS

Создайте или отредактируйте файл:
```bash
mcedit /etc/net/ifaces/ens192/resolv.conf
```
Пример:
```
nameserver 8.8.8.8
```

Перезагрузите сеть:
```bash
systemctl restart network
```

Проверьте доступность DNS:
```bash
ping ya.ru
```
Обновите спискок пакетов и установите всё необходимое командой:
```bash
apt-get update && apt-get install frr dhcp-server wget bind-utils -y
```

#### 6.5 Создание VLAN для офиса HQ - VLAN100 (У ВАС БУДУТ ОТЛИЧАТСЯ VLAN)

0. Создайте папку физического интерфейса к примеру ens224 и внутри создайте options со следущим содержимым:
 ```bash
   BOOTPROTO=static
   TYPE=eth
   ```
1. Создайте каталог для подинтерфейса (замените `<имя_физического_интерфейса>` на фактическое имя, например, `ens224`):
   ```bash
   mkdir /etc/net/ifaces/<имя_физического_интерфейса>.100
   ```
2. Отредактируйте файл настроек:
   ```bash
   mcedit /etc/net/ifaces/<имя_физического_интерфейса>.100/options
   ```
   Пример содержимого:
   ```
   TYPE=vlan
   HOST=ens224
   VID=100
   DISABLED=no
   BOOTPROTO=static
   ONBOOT=yes
   CONFIG_IPV4=yes
   ```
3. Создайте файл для задания IP-адреса:
   ```bash
   mcedit /etc/net/ifaces/<имя_физического_интерфейса>.100/ipv4address
   ```
   Пример:
   ```
   192.168.10.2/26
   ```
У ВАС VLAN Будут ОТЛИЧАТЬСЯ 

Скопируйте файл options из ens224.100 в папку ens224.200 и ens224.999 и вам надо будет просто отредактировать VID=ваш vlan по заданию

#### 6.6 Создание VLAN для офиса HQ – VLAN200

1. Создайте каталог для подинтерфейса:
   ```bash
   mkdir /etc/net/ifaces/<имя_физического_интерфейса>.200
   ```
2. Отредактируйте файл настроек:
   ```bash
   mcedit /etc/net/ifaces/<имя_физического_интерфейса>.200/options
   ```
   Пример содержимого:
   ```
   TYPE=vlan
   HOST=ens224
   VID=200
   DISABLED=no
   BOOTPROTO=static
   ONBOOT=yes
   CONFIG_IPV4=yes
   ```
3. Создайте файл для задания IP-адреса:
   ```bash
   mcedit /etc/net/ifaces/<имя_физического_интерфейса>.200/ipv4address
   ```
   Укажите IP-адрес в формате `ip/mask`.

#### 6.7 Создание VLAN для управления – VLAN999

1. Создайте каталог для подинтерфейса:
   ```bash
   mkdir /etc/net/ifaces/<имя_физического_интерфейса>.999
   ```
2. Отредактируйте файл настроек:
   ```bash
   mcedit /etc/net/ifaces/<имя_физического_интерфейса>.999/options
   
   ```
   Пример содержимого:
   ```
   TYPE=vlan
   HOST=ens224
   VID=999
   DISABLED=no
   BOOTPROTO=static
   ONBOOT=yes
   CONFIG_IPV4=yes
   ```
3. Создайте файл для задания IP-адреса:
   ```bash
   mcedit /etc/net/ifaces/<имя_физического_интерфейса>.999/ipv4address
   ```
   Укажите IP-адрес в формате `ip/mask`.
   
5. Перезагрузите сеть:
   ```bash
   systemctl restart network
   ```
---

### 7. Настройка NAT для офиса HQ

У вас будет другие ПОДСЕТИ 

#### 7.1 Настройка NAT
```bash
iptables -t nat -A POSTROUTING -o ens192 -s 192.168.10.0/26 -j MASQUERADE
```
Сделайте тоже самое для второй сети 

```bash
iptables -t nat -A POSTROUTING -o ens192 -s 192.168.20.0/28 -j MASQUERADE
```
Сохраните правила и перезапустите:
```bash
iptables-save > /etc/sysconfig/iptables
systemctl enable --now iptables
```
7.2 Включение пересылки пакетов

Отредактируйте файл командой ниже по пути `/etc/net/sysctl.conf`:
```bash
mcedit /etc/net/sysctl.conf
```

   - Измените строку net.ipv4.ip_forward:
     ```diff
     -#net.ipv4.ip_forward = 0
     +net.ipv4.ip_forward = 1
     ```
Примените изменения командой:
```bash
systemctl restart network
```
---

### 8. Настройка сети для BR-RTR

<details>
  <summary>Развернуть инструкцию</summary>

#### 8.0 Задание Часового пояса (если не задан)
```bash
timedatectl set-timezone Asia/Novosibirsk
```

#### 8.1 Задание hostname (если не установлен)
```bash
hostnamectl set-hostname br-rtr.au-team.irpo && exec bash
```

#### 8.2 Назначение IP-адреса

Создайте или отредактируйте файл:
```bash
mcedit /etc/net/ifaces/ens192/ipv4address
```
Пример:
```
172.16.5.2/28
```

#### 8.3 Настройка маршрута по умолчанию

Создайте или отредактируйте файл:
```bash
mcedit /etc/net/ifaces/ens192/ipv4route
```
Добавьте:
```
default via 172.16.5.1
```

#### 8.4 Настройка DNS

Создайте или отредактируйте файл:
```bash
mcedit /etc/net/ifaces/ens192/resolv.conf
```
Пример:
```
nameserver 8.8.8.8
```

Перезагрузите сеть:
```bash
systemctl restart network
```

Проверьте доступность DNS:
```bash
ping ya.ru
```
Установите frr:
```bash
apt-get update && apt-get install frr -y
```

#### 8.5 Создание и настройка интерфейса в сторону br-rtr офиса

Создайте или отредактируйте файл options:
```bash
mcedit /etc/net/ifaces/ens224/options
```
Пример:
```
BOOTPROTO=static
TYPE=eth
```

Создайте или отредактируйте файл ipv4address:
```bash
mcedit /etc/net/ifaces/ens224/ipv4address
```
Пример:
```
192.168.30.1/27
```

Перезагрузите сеть:
```bash
systemctl reboot network
```
Проверьте всё ли появилось если нет значит вы косяк))  Ищите проблему и помните что времени всего 2:30 часа)
</details>

---

### 9. Настройка NAT для офиса BR

<details>
  <summary>Развернуть инструкцию</summary>

#### 9.1 Задание IP-адреса для офиса BR

В файле `/etc/net/ifaces/ens224/ipv4address` укажите:
```
192.168.30.1/27
```

#### 9.2 Настройка NAT
У ВАС БУДУТ ДРУГИЕ ПОДСЕТИ
```bash
iptables -t nat -A POSTROUTING -o ens192 -s 192.168.30.0/27 -j MASQUERADE
```

#### 9.3 Сохранение правил и автозапуск
```bash
iptables-save > /etc/sysconfig/iptables
systemctl enable --now iptables
```

#### 9.4 Включение пересылки пакетов

Отредактируйте файл командой ниже по пути `/etc/net/sysctl.conf`:
```bash
mcedit /etc/net/sysctl.conf
```

   - Измените строку net.ipv4.ip_forward:
     ```diff
     -#net.ipv4.ip_forward = 0
     +net.ipv4.ip_forward = 1
     ```
Примените изменения командой:
```bash
systemctl restart network
```

</details>

---

### 10. Создание локальных учётных записей на HQ-RTR

#### 10.1 Создание учётной записи net_admin
```bash
useradd net_admin
```
Задайте пароль:
```bash
passwd net_admin
```
_(Пароль: **P@$$word**)_

#### 10.2 Добавление в группу wheel
```bash
usermod -a -G wheel net_admin
```

#### 10.3 Редактирование файла sudoers

Откройте файл:
```bash
mcedit /etc/sudoers
```
Найдите строку:
```
#WHEEL_USERS ALL=(ALL:ALL) NOPASSWD: ALL
```
Удалите символ `#` и сохраните изменения.

#### 10.4 Проверка работы sudo

Выйдите из root (команда `exit`) и выполните вход под пользователем `net_admin`:
```bash
login: net_admin  
Password: P@$$word
```
Проверьте командой:
```bash
sudo su
```
Успешное выполнение команды подтверждает корректную настройку.

---

### 11. Создание локальных учётных записей на BR-RTR

<details>
  <summary>Развернуть инструкцию</summary>

#### 11.1 Создание учётной записи net_admin
```bash
useradd net_admin
```
Установите пароль:
```bash
passwd net_admin
```
_(Пароль: **P@$$word**)_

#### 11.2 Добавление в группу wheel
```bash
usermod -a -G wheel net_admin
```

#### 11.3 Редактирование файла sudoers

Откройте файл:
```bash
mcedit /etc/sudoers
```
Найдите строку:
```
#WHEEL_USERS ALL=(ALL:ALL) NOPASSWD: ALL
```
Удалите `#` и сохраните изменения.

#### 11.4 Проверка sudo

Выйдите из root и выполните вход под пользователем `net_admin`:
```bash
login: net_admin  
Password: P@$$word
```
Затем выполните:
```bash
sudo su
```
Успешное выполнение команды подтверждает корректную настройку.

</details>

---

### 12. Настройка HQ-SRV

#### 12.0 Задание Часового пояса (если не задан)
```bash
timedatectl set-timezone Asia/Novosibirsk
```

#### 12.1 Задание hostname (если не задан)
```bash
hostnamectl set-hostname hq-srv.au.team.irpo && exec bash
```

---

#### 12.2 Задание IP-адреса

Cоздайте папку саб интерфейса 
```bash
mkdir /etc/net/ifaces/ens192.100/
```
Отредактируйте файл options:
   ```bash
mcedit /etc/net/ifaces/<имя_физического_интерфейса>.100/options
   ```
   Пример содержимого:
   ```
   TYPE=vlan
   HOST=ens192
   VID=100
   DISABLED=no
   BOOTPROTO=static
   ONBOOT=yes
   CONFIG_IPV4=yes
   ```

Создайте или отредактируйте файл:
```bash
mcedit /etc/net/ifaces/ens192.100/ipv4address
```
Пример:
```
192.168.10.2/26
```
#### 12.3 Настройка маршрута по умолчанию

Создайте или отредактируйте файл:
```bash
mcedit /etc/net/ifaces/ens192.100/ipv4route
```
Добавьте:
```
default via 192.168.10.1
```

#### 12.4 Настройка DNS

Создайте или отредактируйте файл:
```bash
mcedit /etc/net/ifaces/ens192.100/resolv.conf
```
Пример:
```
nameserver 8.8.8.8
```
Cоздайте папку саб интерфейса для менаджмент интрефейса 
```bash
mkdir /etc/net/ifaces/ens192.999/
```

Отредактируйте файл options:
   ```bash
mcedit /etc/net/ifaces/<имя_физического_интерфейса>.999/options
   ```
   Пример содержимого:
   ```
   TYPE=vlan
   HOST=ens192
   VID=999
   DISABLED=no
   BOOTPROTO=static
   ONBOOT=yes
   CONFIG_IPV4=yes
   ```

Создайте или отредактируйте файл:
```bash
mcedit /etc/net/ifaces/ens192.999/ipv4address
```
Пример:
```
192.168.99.2/29
```

Перезагрузите сеть:
```bash
systemctl restart network
```

Проверьте доступность DNS:
```bash
ping ya.ru
```

Обновите пакеты и установите всё необходимое командой:
```bash
apt-get update -y && apt-get install bind bind-utils wget -y
```

---

### 13. Создание локальной учётной записи на HQ-SRV

У вас -u будет другой 
#### 13.1 Создание учётной записи sshuser
```bash
useradd -u 1010 sshuser
```
Задайте пароль:
```bash
passwd sshuser
```
_(Пароль: **P@ssw0rd**)_

#### 13.2 Добавление в группу wheel
```bash
usermod -a -G wheel sshuser
```

#### 13.3 Редактирование файла sudoers

Откройте файл:
```bash
mcedit /etc/sudoers
```
Найдите строку:
```
#WHEEL_USERS ALL=(ALL:ALL) NOPASSWD: ALL
```
Удалите символ `#` и сохраните изменения.

#### 13.4 Проверка sudo

Выйдите из root и выполните вход под пользователем `sshuser`:
```bash
login: sshuser
Password: P@ssw0rd
```
Затем выполните:
```bash
sudo su
```
Если команда выполнена успешно – настройка корректна.

#### 13.5 Настройка SSH на HQ-SRV

1. Откройте файл конфигурации SSH:
   ```bash
   mcedit /etc/openssh/sshd_config
   ```
2. Внесите следующие изменения (раскомментируйте и измените значения ПАРАМЕТРЫ МОГУТ БЫТЬ ДРУГИМИ ЭТО ПРИМЕР):
   - Измените порт с 22 на 2024:
     ```diff
     -#Port 22
     +Port 2024
     ```
   - Ограничьте доступ только для пользователя `sshuser`:
     ```diff
     -#Match User anoucvs
     +Match User sshuser
     ```
   - Задайте баннер:
     ```diff
     -#Banner none
     +Banner /etc/openssh/Banner.txt
     ```
   - Ограничьте число попыток аутентификации:
     ```diff
     -#MaxAuthTries 6
     +MaxAuthTries 2
     ```
3. Сохраните изменения (F2, затем F10).

4. Создайте файл баннера:
   ```bash
   mcedit /etc/openssh/Banner.txt
   ```
   Добавьте:
   ```
   Authorized access only
   ```

5. Перезагрузите службу SSH:
   ```bash
   systemctl restart sshd.service
   ```

6. Проверьте все ли правильно настроили подключившись по ssh с hq-rtr:
   ```bash
   ssh sshuser@192.168.10.2 -p 2024
   ```
---

### 14. Настройка BR-SRV

<details>
  <summary>Развернуть инструкцию</summary>


#### 14.0 Задание Часового пояса (если не задан)
```bash
timedatectl set-timezone Asia/Novosibirsk
```

#### 14.1 Задание hostname (если не задан)
```bash
hostnamectl set-hostname br-srv.au.team.irpo && exec bash
```

#### 14.2 Задание IP-адреса

Создайте или отредактируйте файл:
```bash
mcedit /etc/net/ifaces/ens192/ipv4address
```
Пример:
```
192.168.30.2/27
```

#### 14.4 Настройка маршрута по умолчанию

Создайте или отредактируйте файл:
```bash
mcedit /etc/net/ifaces/ens192/ipv4route
```
Добавьте:
```
default via 192.168.30.1
```

#### 14.5 Настройка DNS

Создайте или отредактируйте файл:
```bash
mcedit /etc/net/ifaces/ens192/resolv.conf
```
Пример:
```
nameserver 8.8.8.8
```

Перезагрузите сеть:
```bash
systemctl restart network
```

Проверьте доступность DNS:
```bash
ping ya.ru
```

У вас -u будет другой 
#### 14.6 Создание учётной записи sshuser
```bash
useradd -u 1010 sshuser
```
Задайте пароль:
```bash
passwd sshuser
```
_(Пароль: **P@ssw0rd**)_

#### 16.7 Добавление в группу wheel
```bash
usermod -a -G wheel sshuser
```

#### 14.8 Редактирование файла sudoers

Откройте файл:
```bash
mcedit /etc/sudoers
```
Найдите строку:
```
#WHEEL_USERS ALL=(ALL:ALL) NOPASSWD: ALL
```
Удалите `#` и сохраните изменения.

#### 16.9 Проверка sudo

Выйдите из root и выполните вход под пользователем `sshuser`:
```bash
login: sshuser
Password: P@ssw0rd
```
Затем выполните:
```bash
sudo su
```
Если команда выполнена успешно – настройка корректна.

</details>

---

#### 14.10 Настройка SSH на BR-SRV

<details>
  <summary>Развернуть инструкцию</summary>

1. Откройте файл конфигурации SSH:
   ```bash
   mcedit /etc/openssh/sshd_config
   ```
2. Внесите следующие изменения (раскомментируйте и измените значения ПАРАМЕТРЫ МОГУТ БЫТЬ ДРУГИМИ ЭТО ПРИМЕР):
   - Измените порт с 22 на 2024:
     ```diff
     -#Port 22
     +Port 2024
     ```
   - Ограничьте доступ только для пользователя `sshuser`:
     ```diff
     -#Match User anoucvs
     +Match User sshuser
     ```
   - Задайте баннер:
     ```diff
     -#Banner none
     +Banner /etc/openssh/Banner.txt
     ```
   - Ограничьте число попыток аутентификации:
     ```diff
     -#MaxAuthTries 6
     +MaxAuthTries 2
     ```
3. Сохраните изменения (F2, затем F10).

4. Создайте файл баннера:
   ```bash
   mcedit /etc/openssh/Banner.txt
   ```
   Добавьте:
   ```
   Authorized access only
   ```

5. Перезагрузите службу SSH:
   ```bash
   systemctl restart sshd.service
   ```

6. Проверьте все ли правильно настроили подключившись по ssh с br-rtr:
   ```bash
   ssh sshuser@192.168.30.2 -p 2024
   ```

</details>

---

### 15. Настройка динамической маршрутизации

#### 15.1 Настройка туннеля GRE и OSPF на HQ-RTR

1. **Создание интерфейса туннеля**

   Создайте каталог для интерфейса туннеля:
   ```bash
   mkdir /etc/net/ifaces/gre1
   ```

2. **Настройка файла options для туннеля**

   Создайте и отредактируйте файл:
   ```bash
   mcedit /etc/net/ifaces/gre1/options
   ```
   У вас ip могут отличаться 
   Пример содержимого:
   ```
   TUNLOCAL=172.16.4.2
   TUNREMOTE=172.16.5.2
   TUNTYPE=gre
   TYPE=iptun
   TUNOPTIONS='ttl 64'
   HOST=ens192
   ```

3. **Настройка IP-адреса туннеля**

   Создайте файл:
   ```bash
   mcedit/etc/net/ifaces/gre1/ipv4address
   ```
   Пример:
   ```
   172.16.100.2/29
   ```

4. Перезагрузите сеть и проверьте интерфейс:
   ```bash
   systemctl restart network
   ip a
   ```

5. **Настройка frr**

   Добавьте службу `frr` в автозагрузку:
   ```bash
   systemctl enable --now frr
   ```
   Отредактируйте конфигурационный файл демонов:
   ```bash
   mcedit /etc/frr/daemons
   ```
   - Меняем ospfd:
     ```diff
     -#ospfd=no
     +ospfd=yes
     ```
   Сохраните изменения.
   
   Перезагрузите службу frr:
   ```
   systemctl reboot frr
   ```
5. **Настройка OSPF через vtysh**

   Введите:
   ```bash
   vtysh
   ```
   Комнды для настройки ospf
   ```
   show running-config (чтобы проверить если там что нибудь или нет)
   conf t
   router ospf
     passive interface default
     network 172.16.100.0/29 area 0 сеть тунеля 
     network 192.168.10.0/26 area 0 сеть в сторону hq-srv 
     network 192.168.20.0/28 area 0 сеть в сторону hq-сli
     area 0 authentication
   exit
   interface gre1 тут вы указывете название вашего тунеля между hq-rtr и br-rtr
     no ip ospf passive
     ip ospf authentication-key 1245 ваш ключ  
   exit
   do wr
   end
   exit
   ```
   Перезагрузите сеть:
   ```bash
   systemctl restart network
   ```

#### 15.2 Настройка туннеля GRE и OSPF на BR-RTR

<details>
  <summary>Развернуть инструкцию</summary>

1. **Создание интерфейса туннеля**

   Создайте каталог:
   ```bash
   mkdir /etc/net/ifaces/gre1
   ```

2. **Настройка файла options для туннеля**

   Создайте и отредактируйте файл:
   ```bash
   mcedit /etc/net/ifaces/gre1/options
   ```
   Пример содержимого:
   ```
   TUNLOCAL=172.16.5.2
   TUNREMOTE=172.16.4.2
   TUNTYPE=gre
   TYPE=iptun
   TUNOPTIONS='ttl 64'
   HOST=ens192
   ```

3. **Настройка IP-адреса туннеля**

   Создайте файл:
   ```bash
   mcedit /etc/net/ifaces/gre1/ipv4address
   ```
   Пример:
   ```
   172.16.100.1/29
   ```

4. Перезагрузите сеть:
   ```bash
   systemctl restart network
   ip a
   ```

5. **Настройка frr**

   Добавьте службу `frr` в автозагрузку:
   ```bash
   systemctl enable --now frr
   ```
   Отредактируйте конфигурационный файл демонов:
   ```bash
   mcedit /etc/frr/daemons
   ```
     - Меняем ospfd:
     ```diff
     -#ospfd=no
     +ospfd=yes
     ```
   Сохраните изменения.
   
   Перезагрузите службу frr:
   ```
   systemctl reboot frr
   ```
5. **Настройка OSPF через vtysh**

   Введите:
   ```bash
   vtysh
   ```
   Комнды для настройки ospf
   ```
   show running-config (чтобы проверить если там что нибудь или нет)
   conf t
   router ospf
     passive interface default
     network 172.16.100.0/29 area 0 сеть тунеля 
     network 192.168.30.0/27 area 0 сеть в сторону br-rtr
     area 0 authentication
   exit
   interface gre1 тут вы указывете название вашего тунеля между hq-rtr и br-rtr
     no ip ospf passive
     ip ospf authentication-key 1245 ваш ключ  
   exit
   do wr
   end
   exit
   ```
9. Перезагрузите сеть:
   ```bash
   systemctl restart network
   ```
10. Проверьте настройки:
    ```bash
    vtysh
    show ip ospf neighbor
    ```
    Если сосед отображается – настройка выполнена корректно.

</details>

---

### 16. Настройка DHCP и DNS

#### 16.1 Конфигурация DHCP

1. перейдите в папку `/etc/dhcp/` В ней скачайте файл yNdsd0ur командой ниже:
    ```bash
    wget -O dhcpd.conf gclnk.com/yNdsd0ur
    ```
Отредактируйте файл `dhcpd` под cвою сеть командой:
   ```bash
   mcedit /etc/dhcp/dhcpd.conf
   ```
   После чего идёт в файл `dhcpd` командой:

   ```bash
   mcedit /etc/sysconfig/dhcpd
   ```
Внутри пишем интерфейс который будет слушать клиент например ens224.200 
  
  ```bash
  DHCPDARGS=ens224.200
   ```
Перезагружаем сервер командой:
```bash
 systemctl restart dhcpd
   ```
Добавляем его в авто-загрузку командой:
```bash
  systemctl enable dhcpd
   ```
6. На клиенте (например, HQ-CLI) Отключаем Network-manager если не отлючили в самом начале командой:
   ```bash
    systemctl disable --now Network-manager 
   ```
Теперь идёт в сетевой интерфейс клиента например `ens192` и в файле `options`
   - Меняем NV_CONTROLLED:
     ```diff
     -#NV_CONTROLLED=yes
     +NV_CONTROLLED=no
     ```
     - Меняем DISABLED:
     ```diff
     -#DISABLED=yes
     +DISABLED=no
     ```
 после чего создаём папку саб интерфейса командой
 ```bash
   mkdir /etc/net/ifaces192.200
   ```
И внутри файл options со следущим содержимым

   Пример содержимого:
   ```
   TYPE=vlan
   HOST=ens192
   VID=200
   DISABLED=no
   BOOTPROTO=dhcp
   ONBOOT=yes
   CONFIG_IPV4=yes
   ```
Перезагружаем сеть командой 
```bash
  systemctl restart network
   ```
Если всё работает то должен появится интернет (в файле конфига dhcp поставьте dns сервер гугла если вы не настраивали ещё dns)

#### 19.2 Конфигурация DNS (BIND)
сначала идём в `options` командой:

```bash
  mcedit /etc/bind/options.conf 
   ```
Тут делаем как на картинке
![image](https://github.com/user-attachments/assets/409ae1c6-929b-4acb-9f28-fcb97e90aab1)
После сохраняем перезагружаем `bind` и добавляем его в авто-загрузку командами:
```bash
  systemctl reboot bind
  systemctl enable bind
   ```

 Теперь идёт в интерфейс в котором мы у нас прописан dns-server и меняем:
 
   - Измените nameserver:
     ```diff
     -#nameserver 8.8.8.8
     +nameserver 127.0.0.1
     ```
Перезагружаем сеть командой:
```bash
  systemctl reboot network
   ```
Проверяем командой:
 ```bash
  nslookup ya.ru
   ```
Если всё хорошо и нету ошибок переходим в папку `bind` командой:
 ```bash
  cd/etc/bind
   ```
Скачиваем и копируем файлы в папку `bind` командами:
 ```bash
  wget gclnk.com/p3STJhug
  mv p3STJhug db.192.168.10
   ```

 ```bash
  wget gclnk.com/Z7czmF7J
  mv Z7czmF7J db.192.168.20
   ```

 ```bash
  wget gclnk.com/4emEp6Xo
  mv 4emEp6Xo db.au-team.irpo
   ```

 ```bash
  wget gclnk.com/Ce9hyOKh
  mv Ce9hyOKh local.conf
   ```
После того как всё сделали перезапускаем `bind` комндой:
 
 ```bash
  systemctl reboot bind
   ```
5. Проверьте конфигурацию:
   ```bash
   sudo named-checkconf
   sudo named-checkzone au-team.irpo /etc/bind/db.au-team.irpo
   ```
6. Проверьте работу: У ВАС МОГУТ БЫТЬ РАЗНЫЕ IP 
   ```bash
   nslookup hq-rtr.au-team.irpo 127.0.0.1
   nslookup 192.168.10.1 127.0.0.1
   ```
ПРОВЕРЯЕМ ВСЁ ОТ НАЧАЛО И ДО КОНЦА А КАК Я НЕ СКАЖУ)))) 

ЛАДНО ТАК СКАЖУ

С КЛИЕНТА ПОДКЛЮЧАЕМСЯ К СЕРВЕРАМ ПО SSH ПРОВЕРЯЕМ DHS SERVER И ПРОЧИЕ...
