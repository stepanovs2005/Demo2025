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
| HQ-RTR (VLAN200)  | 192.168.20.1/29                  | 192.168.20.1 |
| HQ-RTR (VLAN999)  | 192.168.99.1/29                  | 192.168.99.1 | 
| HQ-CLI           | 192.168.20.2/29, 192.168.99.2/29  | 192.168.20.1/29,192,168,99,1/29 |
| HQ-SRV           | 192.168.10.2/26                  | 192.168.10.1/26 |
| BR-RTR           | 192.168.30.1/27                  | 172.16.5.2/28 |
| BR-SRV           | 192.168.30.2/27                  | 192.168.30.1/27 |

https://jodies.de/ipcalc - ip-калькулятор

![image](https://github.com/user-attachments/assets/da181112-1d6d-4da9-bc8c-7bacc63d12d5)

---

## Решение

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
hostnamectl set-hostname isp.au-team.irpo && exec bash
```

#### 2.2 Проверка получения настроек DHCP

Проверьте командой:
```bash
ip addr
```
Если настройки не получены – настройте DHCP, отредактировав соответствующий конфигурационный файл (например, `/etc/net/ifaces/ens33/options`).

---

### 3. Перезагрузка сети и обновление пакетов

Перезапуск сетевых сервисов и установка необходимых пакетов:
```bash
systemctl restart network
apt-get update -y && apt-get install nano iptables -y
clear
```

---

### 4. Настройка сетевых адаптеров

1. Определите имя сетевого адаптера командой:
   ```bash
   ip addr
   ```
2. Создайте каталог для интерфейса (например, для `ens33`):
   ```bash
   mkdir -p /etc/net/ifaces/ens33
   ```
3. Отредактируйте файл настроек:
   ```bash
   nano /etc/net/ifaces/ens33/options
   ```
   Пример содержимого:
   ```bash
   BOOTPROTO=static
   TYPE=eth
   ```
4. Создайте файл для задания IP-адреса (например, для подсети HQ):
   ```bash
   nano /etc/net/ifaces/ens33/ipv4address
   ```
   Пример:
   ```
   172.16.4.1/28
   ```
5. Аналогичным образом задайте IP-адреса для других подсетей и перезагрузите сеть:
   ```bash
   systemctl restart network
   ip addr
   ```

---

### 5. Настройка NAT (маскарадинга)

Для обеспечения доступа к Интернету настройте маскарадинг для соответствующих подсетей.

#### 5.1 Пример для HQ-RTR:
```bash
iptables -t nat -A POSTROUTING -o ens33 -s 172.16.4.0/28 -j MASQUERADE
```

#### 5.2 Пример для BR-RTR:
```bash
iptables -t nat -A POSTROUTING -o ens33 -s 172.16.5.0/28 -j MASQUERADE
```

Сохраните правила:
```bash
iptables-save > /etc/sysconfig/iptables
systemctl enable --now iptables
```

---

### 6. Включение пересылки пакетов

Отредактируйте файл `/etc/net/sysctl.conf`:
```bash
nano /etc/net/sysctl.conf
```
Измените строку:
```
net.ipv4.ip_forward = 0
```
на:
```
net.ipv4.ip_forward = 1
```
Примените изменения:
```bash
systemctl restart network
```

---

### 7. Настройка сети для HQ-RTR

#### 7.1 Задание hostname (если не задан)
```bash
hostnamectl set-hostname hq-rtr.au-team.irpo && exec bash
```
#### 7.2 Конфигурация интерфейса

Отредактируйте файл `/etc/net/ifaces/ens33/options` и удалите лишние строки (например, `NM_CONTROLLED`, `DISABLED`, `SYSTEMD_CONTROLLED`).

#### 7.3 Задание IP-адреса

Создайте или отредактируйте файл:
```bash
nano /etc/net/ifaces/ens33/ipv4address
```
Пример:
```
172.16.4.2/28
```

#### 7.4 Настройка маршрута по умолчанию

Создайте или отредактируйте файл:
```bash
nano /etc/net/ifaces/ens33/ipv4route
```
Пример:
```
default via 172.16.4.1
```

#### 7.5 Настройка DNS

Создайте или отредактируйте файл:
```bash
nano /etc/net/ifaces/ens33/resolv.conf
```
Пример:
```
nameserver 172.16.4.1
```

удалите `systemd-networkd`:
```bash
apt-get remove systemd-networkd
```

Перезагрузите сеть:
```bash
systemctl restart network
```

Проверьте доступность DNS:
```bash
ping ya.ru
```

---

### 8. Настройка NAT для офиса HQ

#### 8.1 Задание IP-адреса для офиса HQ
Сначала Отредактируйте файл настроек:
   ```bash
   nano /etc/net/ifaces/ens33/options
   ```
   Пример содержимого:
   ```bash
   BOOTPROTO=static
   TYPE=eth
   ```

В файле `/etc/net/ifaces/ens33/ipv4address` укажите:
```
192.168.10.1/26
```

#### 8.2 Настройка NAT
```bash
iptables -t nat -A POSTROUTING -o ens33 -s 192.168.10.0/26 -j MASQUERADE
```

Сохраните правила и перезапустите:
```bash
iptables-save > /etc/sysconfig/iptables
systemctl enable --now iptables
systemctl restart network
```

---

### 9. Настройка сети для BR-RTR

<details>
  <summary>Развернуть инструкцию</summary>

#### 9.1 Задание hostname (если не установлен)
```bash
hostnamectl set-hostname br-rtr.au-team.irpo && exec bash
```

#### 9.2 Конфигурация интерфейса

Откройте файл настроек для `ens33`:
```bash
vi /etc/net/ifaces/ens33/options
```
Удалите строки:
- NM_CONTROLLED
- DISABLED
- SYSTEMD_CONTROLLED

#### 9.3 Назначение IP-адреса

Создайте или отредактируйте файл:
```bash
vi /etc/net/ifaces/ens33/ipv4address
```
Пример:
```
172.16.5.2/28
```

#### 9.4 Настройка маршрута по умолчанию

Создайте или отредактируйте файл:
```bash
vi /etc/net/ifaces/ens33/ipv4route
```
Добавьте:
```
default via 172.16.5.1
```

#### 9.5 Настройка DNS

Создайте или отредактируйте файл:
```bash
vi /etc/net/ifaces/ens33/resolv.conf
```
Пример:
```
nameserver 172.16.5.1
```

Перезагрузите сеть:
```bash
systemctl restart network
```

Проверьте доступность DNS:
```bash
ping ya.ru
```

Обновите систему и установите nano:
```bash
apt-get update -y && apt-get install nano -y
```

</details>

---

### 10. Настройка NAT для офиса BR

<details>
  <summary>Развернуть инструкцию</summary>

#### 10.1 Задание IP-адреса для офиса BR

В файле `/etc/net/ifaces/ens33/ipv4address` укажите:
```
192.168.30.1/27
```

#### 10.2 Настройка NAT
```bash
iptables -t nat -A POSTROUTING -o ens33 -s 192.168.30.0/27 -j MASQUERADE
```

#### 10.3 Сохранение правил и автозапуск
```bash
iptables-save > /etc/sysconfig/iptables
systemctl enable --now iptables
```

#### 10.4 Включение пересылки пакетов

Убедитесь, что в файле `/etc/net/sysctl.conf` установлено:
```
net.ipv4.ip_forward = 1
```
и перезагрузите сеть:
```bash
systemctl restart network
```

</details>

---

### 11. Создание локальных учётных записей на HQ-RTR

#### 11.1 Создание учётной записи net_admin
```bash
useradd net_admin
```
Задайте пароль:
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
nano /etc/sudoers
```
Найдите строку:
```
#WHEEL_USERS ALL=(ALL:ALL) NOPASSWD: ALL
```
Удалите символ `#` и сохраните изменения.

#### 11.4 Проверка работы sudo

Выйдите из root (команда `exit`) и выполните вход под пользователем `net_admin`:
```bash
login: net_admin  
Password: P@$$word
```
Проверьте командой:
```bash
sudo su
```
Если приглашение изменилось, настройка выполнена корректно.

---

### 12. Создание локальных учётных записей на BR-RTR

<details>
  <summary>Развернуть инструкцию</summary>

#### 12.1 Создание учётной записи net_admin
```bash
useradd net_admin
```
Установите пароль:
```bash
passwd net_admin
```
_(Пароль: **P@$$word**)_

#### 12.2 Добавление в группу wheel
```bash
usermod -a -G wheel net_admin
```

#### 12.3 Редактирование файла sudoers

Откройте файл:
```bash
nano /etc/sudoers
```
Найдите строку:
```
#WHEEL_USERS ALL=(ALL:ALL) NOPASSWD: ALL
```
Удалите `#` и сохраните изменения.

#### 12.4 Проверка sudo

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

### 13. Настройка VLAN

#### 13.1 Создание VLAN для офиса HQ – VLAN100

1. Создайте каталог для подинтерфейса (замените `<имя_физического_интерфейса>` на фактическое имя, например, `ens34`):
   ```bash
   mkdir -p /etc/net/ifaces/<имя_физического_интерфейса>.hq.100
   ```
2. Отредактируйте файл настроек:
   ```bash
   nano /etc/net/ifaces/<имя_физического_интерфейса>.hq.100/options
   ```
   Пример содержимого:
   ```
   TYPE=vlan
   HOST=ens34
   VID=100
   DISABLED=no
   BOOTPROTO=static
   ONBOOT=yes
   CONFIG_IPV4=yes
   ```
3. Создайте файл для задания IP-адреса:
   ```bash
   nano /etc/net/ifaces/<имя_физического_интерфейса>.hq.100/ipv4address
   ```
   Пример:
   ```
   192.168.10.2/26
   ```

#### 13.2 Создание VLAN для офиса HQ – VLAN200

1. Создайте каталог для подинтерфейса:
   ```bash
   mkdir -p /etc/net/ifaces/<имя_физического_интерфейса>.hq.200
   ```
2. Отредактируйте файл настроек:
   ```bash
   nano /etc/net/ifaces/<имя_физического_интерфейса>.hq.200/options
   ```
   Пример содержимого:
   ```
   TYPE=vlan
   HOST=ens34
   VID=200
   DISABLED=no
   BOOTPROTO=static
   ONBOOT=yes
   CONFIG_IPV4=yes
   ```
3. Создайте файл для задания IP-адреса:
   ```bash
   nano /etc/net/ifaces/<имя_физического_интерфейса>.hq.200/ipv4address
   ```
   Укажите IP-адрес в формате `ip/mask`.

#### 13.3 Создание VLAN для управления – VLAN999

1. Создайте каталог для подинтерфейса:
   ```bash
   mkdir -p /etc/net/ifaces/<имя_физического_интерфейса>.hq.999
   ```
2. Отредактируйте файл настроек:
   ```bash
   nano /etc/net/ifaces/<имя_физического_интерфейса>.hq.999/options
   ```
   Пример содержимого:
   ```
   TYPE=vlan
   HOST=ens34
   VID=999
   DISABLED=no
   BOOTPROTO=static
   ONBOOT=yes
   CONFIG_IPV4=yes
   ```
3. Создайте файл для задания IP-адреса:
   ```bash
   nano /etc/net/ifaces/<имя_физического_интерфейса>.hq.999/ipv4address
   ```
   Укажите IP-адрес в формате `ip/mask`.
4. Перезагрузите сеть:
   ```bash
   systemctl restart network
   ```

#### 13.4 Настройка VLAN на BR-RTR (для VLAN999)

<details>
  <summary>Развернуть инструкцию</summary>
  
1. Создайте каталог для подинтерфейса (замените `<имя_физического_интерфейса>`, например, на `ens33`):
   ```bash
   mkdir -p /etc/net/ifaces/<имя_физического_интерфейса>.br.999
   ```
2. Отредактируйте файл настроек:
   ```bash
   nano /etc/net/ifaces/<имя_физического_интерфейса>.br.999/options
   ```
   Пример содержимого:
   ```
   TYPE=vlan
   HOST=ens33
   VID=999
   DISABLED=no
   BOOTPROTO=static
   ONBOOT=yes
   CONFIG_IPV4=yes
   ```
3. Создайте файл для задания IP-адреса:
   ```bash
   nano /etc/net/ifaces/<имя_физического_интерфейса>.br.999/ipv4address
   ```
   Укажите IP-адрес в формате `ip/mask`.
4. Перезагрузите сеть:
   ```bash
   systemctl restart network
   ```
  
</details>

---

### 14. Настройка HQ-SRV

#### 14.1 Задание hostname (если не задан)
```bash
hostnamectl set-hostname hq-srv.au.team.irpo && exec bash
```

#### 14.2 Настройка интерфейса

Откройте файл настроек для `ens33`:
```bash
vi /etc/net/ifaces/ens33/options
```
Удалите (если присутствуют) строки:  
- NM_CONTROLLED  
- DISABLED  
- SYSTEMD_CONTROLLED

#### 14.3 Задание IP-адреса

Создайте или отредактируйте файл:
```bash
vi /etc/net/ifaces/ens33/ipv4address
```
Пример:
```
192.168.10.2/26
```

#### 14.4 Настройка маршрута по умолчанию

Создайте или отредактируйте файл:
```bash
vi /etc/net/ifaces/ens33/ipv4route
```
Добавьте:
```
default via 192.168.10.1
```

#### 14.5 Настройка DNS

Создайте или отредактируйте файл:
```bash
vi /etc/net/ifaces/ens33/resolv.conf
```
Пример:
```
nameserver 192.168.10.1
```

удалите `systemd-networkd`:
```bash
apt-get remove systemd-networkd
```

Перезагрузите сеть:
```bash
systemctl restart network
```

Проверьте доступность DNS:
```bash
ping ya.ru
```

Обновите пакеты и установите nano:
```bash
apt-get update -y && apt-get install nano -y
```

---

### 15. Создание локальных учётных записей на HQ-SRV

#### 15.1 Создание учётной записи sshuser
```bash
useradd -u 1010 sshuser
```
Задайте пароль:
```bash
passwd sshuser
```
_(Пароль: **P@ssw0rd**)_

#### 15.2 Добавление в группу wheel
```bash
usermod -a -G wheel sshuser
```

#### 15.3 Редактирование файла sudoers

Откройте файл:
```bash
nano /etc/sudoers
```
Найдите строку:
```
#WHEEL_USERS ALL=(ALL:ALL) NOPASSWD: ALL
```
Удалите символ `#` и сохраните изменения.

#### 15.4 Проверка sudo

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

---

### 16. Настройка BR-SRV

<details>
  <summary>Развернуть инструкцию</summary>

#### 16.1 Задание hostname (если не задан)
```bash
hostnamectl set-hostname br-srv.au.team.irpo && exec bash
```

#### 16.2 Настройка интерфейса

Откройте файл для `ens33`:
```bash
vi /etc/net/ifaces/ens33/options
```
Удалите строки:
- NM_CONTROLLED  
- DISABLED  
- SYSTEMD_CONTROLLED

#### 16.3 Задание IP-адреса

Создайте или отредактируйте файл:
```bash
vi /etc/net/ifaces/ens33/ipv4address
```
Пример:
```
192.168.30.2/27
```

#### 16.4 Настройка маршрута по умолчанию

Создайте или отредактируйте файл:
```bash
vi /etc/net/ifaces/ens33/ipv4route
```
Добавьте:
```
default via 192.168.30.1
```

#### 16.5 Настройка DNS

Создайте или отредактируйте файл:
```bash
vi /etc/net/ifaces/ens33/resolv.conf
```
Пример:
```
nameserver 192.168.30.1
```

При необходимости удалите `systemd-networkd`:
```bash
apt-get remove systemd-networkd
```

Перезагрузите сеть:
```bash
systemctl restart network
```

Проверьте доступность DNS:
```bash
ping ya.ru
```

Обновите систему и установите nano:
```bash
apt-get update -y && apt-get install nano -y
```

#### 16.6 Создание учётной записи sshuser
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

#### 16.8 Редактирование файла sudoers

Откройте файл:
```bash
nano /etc/sudoers
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

### 17. Настройка безопасного удалённого доступа на серверах HQ-SRV и BR-SRV

#### 17.1 Настройка SSH на HQ-SRV

1. Откройте файл конфигурации SSH:
   ```bash
   nano /etc/openssh/sshd_config
   ```
2. Внесите следующие изменения (раскомментируйте и измените значения):
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
     +Banner /etc/openssh/sshd_config/Banner.txt
     ```
   - Ограничьте число попыток аутентификации:
     ```diff
     -#MaxAuthTries 6
     +MaxAuthTries 2
     ```
3. Сохраните изменения (Ctrl+O, затем Ctrl+X).

4. Создайте файл баннера:
   ```bash
   nano /etc/openssh/sshd_config/Banner.txt
   ```
   Добавьте:
   ```
   Authorized access only
   ```

5. Перезагрузите службу SSH:
   ```bash
   systemctl restart sshd.service
   ```

#### 17.2 Настройка SSH на BR-SRV

<details>
  <summary>Развернуть инструкцию</summary>

1. Откройте файл конфигурации:
   ```bash
   nano /etc/openssh/sshd_config
   ```
2. Внесите следующие изменения:
   - Измените порт:
     ```diff
     -#Port 22
     +Port 2024
     ```
   - Ограничьте доступ для пользователя:
     ```diff
     -#Match User anoucvs
     +Match User sshuser
     ```
   - Задайте баннер:
     ```diff
     -#Banner none
     +Banner /etc/openssh/sshd_config/Banner.txt
     ```
   - Ограничьте число попыток аутентификации:
     ```diff
     -#MaxAuthTries 6
     +MaxAuthTries 2
     ```
3. Сохраните файл (Ctrl+O, затем Ctrl+X).

4. Создайте или отредактируйте баннер:
   ```bash
   nano /etc/openssh/sshd_config/Banner.txt
   ```
   Добавьте:
   ```
   Authorized access only
   ```

5. Перезагрузите службу SSH:
   ```bash
   systemctl restart sshd.service
   ```

</details>

---

### 18. Настройка динамической маршрутизации

#### 18.1 Настройка туннеля GRE и OSPF на HQ-RTR

1. **Создание интерфейса туннеля**

   Создайте каталог для интерфейса туннеля:
   ```bash
   mkdir -p /etc/net/ifaces/gre1
   ```

2. **Настройка файла options для туннеля**

   Создайте и отредактируйте файл:
   ```bash
   vi /etc/net/ifaces/gre1/options
   ```
   Пример содержимого:
   ```
   TUNLOCAL=172.16.4.2
   TUNREMOTE=172.16.5.2
   TUNTYPE=gre
   TYPE=iptun
   TUNTTL=64
   TUNMTU=1476
   TUNOPTIONS='ttl 64'
   ```

3. **Настройка IP-адреса туннеля**

   Создайте файл:
   ```bash
   vi /etc/net/ifaces/gre1/ipv4address
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

https://www.cisco.com/c/en/us/support/docs/ip/open-shortest-path-first-ospf/13697-25.html

5. **Установка и настройка OSPF**

   Установите пакет `frr`:
   ```bash
   apt-get install frr -y
   ```
   Добавьте службу `frr` в автозагрузку:
   ```bash
   systemctl enable --now iptables
   ```
   Отредактируйте конфигурационный файл демонов:
   ```bash
   nano /etc/frr/demons
   ```
   Найдите строку:
   ```
   ospfd=no
   ```
   Измените на:
   ```
   ospfd=yes
   ```
   Сохраните изменения.

6. **Настройка OSPF через vtysh**

   Введите:
   ```bash
   vtysh
   ```
   Внутри выполните следующие команды:
   ```
   show running-config    # Просмотр текущей конфигурации (при необходимости удалите ненужные интерфейсы)
   conf t
   interface <название_интерфейса_если_не_нужен>
     shutdown
   exit
   router ospf
     ospf router id 172.16.4.1
     passive interface default
     network 172.16.100.0/29 area 0
     network 172.16.10.0/26 area 0
     network 172.16.4.0/28 area 0
   exit
   interface gre1
     no ip ospf passive
   exit
   do wr
   end
   exit
   ```
   Перезагрузите сеть:
   ```bash
   systemctl restart network
   ```
   Проверьте соседей:
   ```bash
   vtysh
   show ip ospf neighbor
   ```

#### 18.2 Настройка туннеля GRE и OSPF на BR-RTR

<details>
  <summary>Развернуть инструкцию</summary>

1. **Создание интерфейса туннеля**

   Создайте каталог:
   ```bash
   mkdir -p /etc/net/ifaces/gre1
   ```

2. **Настройка файла options для туннеля**

   Создайте и отредактируйте файл:
   ```bash
   vi /etc/net/ifaces/gre1/options
   ```
   Пример содержимого:
   ```
   TUNLOCAL=172.16.5.2
   TUNREMOTE=172.16.4.2
   TUNTYPE=gre
   TYPE=iptun
   TUNTTL=64
   TUNMTU=1476
   TUNOPTIONS='ttl 64'
   ```

3. **Настройка IP-адреса туннеля**

   Создайте файл:
   ```bash
   vi /etc/net/ifaces/gre1/ipv4address
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

5. **Установка OSPF**

   Установите пакет:
   ```bash
   apt-get install frr -y
   ```
6. Добавьте службу `frr` в автозагрузку:
   ```bash
   systemctl enable --now iptables
   ```
7. Отредактируйте файл демонов:
   ```bash
   nano /etc/frr/demons
   ```
   Измените:
   ```
   ospfd=no
   ```
   на:
   ```
   ospfd=yes
   ```
8. **Настройка OSPF через vtysh**

   Введите:
   ```bash
   vtysh
   ```
   Затем выполните следующие команды:
   ```
   show running-config
   conf t
   interface <название_интерфейса_если_не_нужен>
     shutdown
   exit
   router ospf
     ospf router id 172.16.5.1
     passive interface default
     network 172.16.100.0/29 area 0
     network 172.16.30.0/27 area 0
   exit
   interface gre1
     no ip ospf passive
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

### 19. Настройка DNS и DHCP

#### 19.1 Конфигурация DNS (BIND)

gclnk.com/p3STJhug db.192.168.10

gclnk.com/Z7czmF7J db.192.168.20

gclnk.com/nRdfbAOr db.192.168.30

gclnk.com/4emEp6Xo db.au-team.irpo

gclnk.com/RnsyEXIX named.conf

gclnk.com/Ce9hyOKh named.conf.local

gclnk.com/shqRkehF named.conf.options


1. Создайте файл `/etc/bind/named.conf` (если отсутствует) и подключите остальные файлы:
   ```bash
   include "/etc/bind/named.conf.options";
   include "/etc/bind/named.conf.local";
   include "/etc/bind/named.conf.default-zones";
   ```
2. Создайте файл `/etc/bind/named.conf.options`:
   ```bash
   nano /etc/bind/named.conf.options
   ```
   Пример содержимого:
   ```conf
   options {
       directory "/var/cache/bind";

       // Публичные DNS для пересылки
       forwarders {
           8.8.8.8;
           8.8.4.4;
       };
       forward first;

       // Включить (или авто) DNSSEC, если требуется
       dnssec-validation auto;

       // Слушать IPv4/IPv6
       listen-on-v6 { any; };

       // При необходимости можно настроить ACL для локальных сетей:
       // acl "hq-network" {
       //     192.168.10.0/26;
       //     192.168.20.0/28;
       //     192.168.30.0/27;
       // };
       // allow-query     { hq-network; };
       // allow-recursion { hq-network; };
   };
   ```
3. Создайте файл `/etc/bind/named.conf.local`:
   ```bash
   nano /etc/bind/named.conf.local
   ```
   Пример содержимого:
   ```conf
   // Зона прямого просмотра (A-записи)
   zone "au-team.irpo" {
       type master;
       file "/etc/bind/db.au-team.irpo";
   };

   // Зона обратного просмотра для сети 192.168.10.0/26
   zone "10.168.192.in-addr.arpa" {
       type master;
       file "/etc/bind/db.192.168.10";
   };

   // Зона обратного просмотра для сети 192.168.20.0/28
   zone "20.168.192.in-addr.arpa" {
       type master;
       file "/etc/bind/db.192.168.20";
   };

   // Зона обратного просмотра для сети 192.168.30.0/27
   zone "30.168.192.in-addr.arpa" {
       type master;
       file "/etc/bind/db.192.168.30";
   };
   ```
4. Создайте файлы зоны:
   - Файл зоны прямого просмотра: `/etc/bind/db.au-team.irpo`
     ```conf
     $TTL 86400
     @   IN  SOA hq-srv.au.team.irpo. root.au-team.irpo. (
             2025031901 ; serial (YYYYMMDDXX)
             3600       ; refresh
             1800       ; retry
             604800     ; expire
             86400 )    ; minimum

         IN  NS  hq-srv.au.team.irpo.

     // A-записи
     hq-rtr   IN  A   192.168.10.1
     br-rtr   IN  A   192.168.30.1
     hq-srv   IN  A   192.168.10.2
     hq-cli   IN  A   192.168.20.2
     br-srv   IN  A   192.168.30.2

     // CNAME-записи
     moodle   IN  CNAME hq-rtr.au-team.irpo.
     wiki     IN  CNAME hq-rtr.au-team.irpo.
     ```
   - Файл зоны обратного просмотра для 192.168.10.0/26: `/etc/bind/db.192.168.10`
     ```conf
     $TTL 86400
     @   IN  SOA hq-srv.au.team.irpo. root.au-team.irpo. (
             2025031901
             3600
             1800
             604800
             86400 )

         IN  NS  hq-srv.au.team.irpo.

     // hq-rtr 192.168.10.1 → PTR
     1   IN  PTR  hq-rtr.au-team.irpo.
     // hq-srv 192.168.10.2 → PTR
     2   IN  PTR  hq-srv.au-team.irpo.
     ```
   - Файл зоны обратного просмотра для 192.168.20.0/28: `/etc/bind/db.192.168.20`
     ```conf
     $TTL 86400
     @   IN  SOA hq-srv.au.team.irpo. root.au-team.irpo. (
             2025031901
             3600
             1800
             604800
             86400 )

         IN  NS  hq-srv.au.team.irpo.

     // hq-cli 192.168.20.2 → PTR
     2   IN  PTR  hq-cli.au-team.irpo.
     ```
   - Файл зоны обратного просмотра для 192.168.30.0/27: `/etc/bind/db.192.168.30`
     ```conf
     $TTL 86400
     @   IN  SOA hq-srv.au.team.irpo. root.au-team.irpo. (
             2025031901
             3600
             1800
             604800
             86400 )

         IN  NS  hq-srv.au.team.irpo.

     // br-rtr 192.168.30.1 → PTR
     1   IN  PTR  br-rtr.au-team.irpo.
     // br-srv 192.168.30.2 → PTR
     2   IN  PTR  br-srv.au-team.irpo.
     ```
5. Проверьте конфигурацию:
   ```bash
   sudo named-checkconf
   sudo named-checkzone au-team.irpo /etc/bind/db.au-team.irpo
   sudo systemctl restart bind9
   sudo systemctl enable bind9
   ```
6. Проверьте работу:
   ```bash
   nslookup hq-rtr.au-team.irpo 127.0.0.1
   nslookup 192.168.10.1 127.0.0.1
   ```


gclnk.com/yNdsd0ur dhcpd.conf

#### 19.2 Конфигурация DHCP

1. Обновите список пакетов и установите `isc-dhcp-server`:
   ```bash
   sudo apt-get update -y
   sudo apt-get install isc-dhcp-server -y
   ```
2. Отредактируйте файл `/etc/dhcp/dhcpd.conf`:
   ```bash
   nano /etc/dhcp/dhcpd.conf
   ```
   Пример минимальной конфигурации (для сети VLAN200, 192.168.20.0/28, где HQ-RTR имеет IP 192.168.20.1):
   ```conf
   default-lease-time 600;
   max-lease-time 7200;

   option domain-name "au-team.irpo";
   option domain-name-servers 192.168.10.2;   # HQ-SRV (DNS)
   authoritative;

   subnet 192.168.20.0 netmask 255.255.255.240 {
       range 192.168.20.2 192.168.20.14;
       option routers 192.168.20.1;
   }
   ```
3. Укажите интерфейс для DHCP в файле `/etc/default/isc-dhcp-server` (например, если VLAN200 поднят как `ens33.200`):
   ```conf
   INTERFACESv4="ens33.200"
   INTERFACESv6=""
   ```
   (В RHEL/ALT файл: `/etc/sysconfig/dhcpd` с переменной `DHCPD_IFACES="ens33.200"`. )
4. Запустите и включите сервис:
   ```bash
   sudo systemctl enable isc-dhcp-server
   sudo systemctl start isc-dhcp-server
   ```
5. Для проверки просмотрите логи:
   ```bash
   sudo journalctl -u isc-dhcp-server
   grep dhcp /var/log/syslog
   ```
6. На клиенте (например, HQ-CLI) обновите DHCP-клиент:
   ```bash
   sudo dhclient -r
   sudo dhclient
   ip addr
   ```
   Клиент должен получить адрес из диапазона 192.168.20.2–192.168.20.14, шлюз 192.168.20.1 и DNS 192.168.10.2.
