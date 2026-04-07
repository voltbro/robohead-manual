---
id: 20-сonfiguring-communication
slug: сonfiguring-communication
title: "Настройка коммуникации между устройствами"
sidebar_label: "Настройка коммуникации между устройствами"
sidebar_position: 2
description: "Настройка коммуникации между устройствамиr"
---

# Настройка коммуникации между устройствами

**Стартовые параметры окружения терминала и ROS**

В стандартном режиме Робоголова запускается как самостоятельное устройство, все ROS-сущности запускаются на локальном компьютере устройства (на Raspberry Pi). В этот момент с Робоголовой уже можно работать.

Если вам необходимо настроить взаимодействие с образовательным роботом TurtleBro, то проделайте следующие шаги:

## 1. Подключите оба устройства к одной Wi-Fi сети

Чтобы настроить взаимодействие между устройствами, они должны находиться в одной сети, как и компьютер, с которого будет производиться работа.

## 2. Узнайте IP-адреса устройств

Вы можете воспользоваться разделом [Определение IP-адреса Робоголовы](../20-getting-started/30-connecting-to-device/31-connecting-via-ssh.md#определение-ip-адреса-робоголовы) или инструкцией к роботу TurtleBro: [Как определить IP-адрес робота](https://ros1.turtlebro.ru/pervoe-vklyuchenie-i-nastroika-robota/ssh.html).

Для дальнейшей инструкции примем, что следующие IP-адреса устройств:

| Устройство             | IP-адрес       |
| ---------------------- | -------------- |
| Робот TurtleBro        | `172.20.10.9`  |
| Робоголова Bbrain      | `172.20.10.10` |


## 3. Подключитесь к Робоголове по SSH

```bash
ssh pi@<IP-адрес_Робоголовы>
# Пароль по умолчанию: brobro
```

## 4. Остановите ubuntu-сервис

```bash
sudo systemctl stop robohead.service
```

## 5. Настройка файла `.bashrc`

1. Откройте файл:
   ```bash
   sudo nano ~/.bashrc
   ```
2. Перейдите в конец файла и найдите блок:
   ```bash
   # for work with another robot
   # ip of another robot:
   #export ROS_MASTER_URI=http://192.168.1.73:11311
   # ip of robohead:
   #export ROS_HOSTNAME=192.168.1.53

   # for local work (without a robot)
   export ROS_MASTER_URI=http://127.0.0.1:11311
   export ROS_HOSTNAME=127.0.0.1
   ```
3. Закомментируйте локальные настройки:
   ```bash
   #export ROS_MASTER_URI=http://127.0.0.1:11311
   #export ROS_HOSTNAME=127.0.0.1
   ```
4. Раскомментируйте настройки для взаимодействия и укажите правильные адреса:
   ```bash
   export ROS_MASTER_URI=http://172.20.10.9:11311
   export ROS_HOSTNAME=172.20.10.10
   ```
5. Сохраните и выйдите: `CTRL+S`, `CTRL+X`

## 6. Настройка файла `start.sh`

1. Откройте файл:
   ```bash
   sudo nano ~/start.sh
   ```
2. Примените те же изменения, что и в `.bashrc`, внутри скрипта:
   ```bash
   #!/bin/bash
   source /opt/ros/noetic/setup.bash
   source /home/pi/robohead_ws/devel/setup.bash

   export PYTHONPATH=$PYTHONPATH:$ROS_ROOT/core/roslib/src
   export PYTHONPATH=$PYTHONPATH:/usr/bin/python3
   export PYTHONPATH=$PYTHONPATH:/home/pi/.local/lib/python3.8/site-packages

   # for work with another robot
   export ROS_MASTER_URI=http://172.20.10.9:11311
   export ROS_HOSTNAME=172.20.10.10

   # for local work (without a robot)
   #export ROS_MASTER_URI=http://127.0.0.1:11311
   #export ROS_HOSTNAME=127.0.0.1

   roslaunch robohead_controller robohead_controller_py.launch
   ```
3. Сохраните и выйдите: `CTRL+S`, `CTRL+X`

## 7. Синхронизация данных и перезагрузка устройства

```bash
sudo sync
sudo reboot
```

> **Примечание:** после этого Робоголова будет загружаться только если TurtleBro работает и находится в той же Wi-Fi сети.

## 8. Использование пакета `turtlebro_voice_nav`

Для взаимодействия устройств можно использовать пакет [turtlebro_voice_nav](https://github.com/NikolayIvanovWS/turtlebro_voice_nav).
