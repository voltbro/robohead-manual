---
id: 48-sensor-driver
slug: sensor_driver
title: "Пакет sensor_driver"
sidebar_label: "sensor_driver"
description: "sensor_driver"
draft: true
---

# Пакет `sensor_driver`

**Назначение:** Пакет `sensor_driver` позволяет взаимодействовать с датчиком тока INA219

---

## Содержание пакета

```text
sensor_driver/
├── CMakeLists.txt
├── package.xml
├── setup.py
├── launch/
│   └── sensor_driver.launch
├── config/
│   └── sensor_driver.yaml
├── scripts/
│   └── main.py
│   └── sensor_driver_dependencies/
│     └── INA219.py
└── examples/
    └── example_subscriber.py
```

---

## Запуск пакета

- Пакет `sensor_driver` запускается автоматически при старте устройства.
- Запуск инициируется через launch-файл `robohead_controller_py.launch` из пакета `robohead_controller`.

---

## Основные возможности

Пакет `sensor_driver` взаимодействует с датчиком **INA219** и публикует данные состояние батареи единым сообщением:

* **Топик:** `/robohead_controller/sensor_driver/bat`
* **Сообщение:** `sensor_msgs/BatteryState`
* Поля сообщения:

| Поле         | Описание                                |
| ------------ | --------------------------------------- |
| `voltage`    | Напряжение на батарее, в вольтах           |
| `current`    | Ток через батарею (> 0 при зарядке, < 0 при разрядке), в амперах                 |

## Режим отладки

В режиме отладки пакет `sensor_driver` запускается изолированно (отдельно) для тестирования функций, без участия других компонентов системы.

### Шаг 1. Остановка всех запущенных пакетов

Остановите фоновый Linux-сервис:

```bash
sudo systemctl stop robohead.service
```

---

### Шаг 2. Запуск пакета вручную
Запустите пакет отдельно через launch-файл:
```bash
roslaunch sensor_driver sensor_driver.launch 
```

---

### Шаг 3. Особенности работы в режиме отладки

- **Пространство имен**: топик пакета **не имеет приставки** `/robohead_controller/`. Используется `/sensor_driver/bat` вместо `/robohead_controller/sensor_driver/bat`

- **Файл конфигурации**: настройки берутся из `sensor_driver/config/sensor_driver.yaml` вместо `robohead_controller/config/sensor_driver.yaml`

### Шаг 4. Возможности тестирования

#### Пример работы с топиком на Python

Команда для запуска примера с использованием пакета `sensor_driver` на Python, в котором на экран будут выводиться текущее напряжение и ток аккумулятора в течение 5 секунд:

```bash
# Запустите в отдельном терминале
rosrun sensor_driver example_subscriber.py
```

