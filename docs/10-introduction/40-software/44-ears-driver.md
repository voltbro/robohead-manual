---
id: 44-ears-driver
slug: ears_driver
title: "Пакет ears_driver"
sidebar_label: "ears_driver"
description: "ears_driver"
draft: true
---

# Пакет `ears_driver` 

**Назначение:** Пакет `ears_driver` позволяет взаимодействовать с ушами Робоголовы: плавно перемещать их вперед и назад.

---

## Содержание пакета

```text
ears_driver/
├── CMakeLists.txt
├── package.xml
├── launch/
│   └── ears_driver_py.launch
├── config/
│   └── ears_driver.yaml
├── scripts/
│   └── main.py
├── srv/
│   └── EarsSetAngle.srv
└── examples/
    └── example_zigzag.py
```

---

## Запуск пакета

- Пакет `ears_driver` запускается автоматически при старте устройства.
- Запуск инициируется через launch-файл `robohead_controller_py.launch` из пакета `robohead_controller`.

---

## Основные возможности

Пакет `ears_driver` предоставляет **ROS-сервис `EarsSetAngle`** для управления ушами головы:

**ROS-сервис `EarsSetAngle`** позволяет задать позицию сервоприводов ушей.

   **Пример вызова:**

   ```bash
rosservice call robohead_controller/ears_driver/EarsSetAngle "left_ear_angle: 30
right_ear_angle: 30" 
   ```
Убедитесь, что вы находитесь в режиме отладки!


**Параметры:**
| Параметр           | Описание                                                                 |
|--------------------|--------------------------------------------------------------------------|
| `left_ear_angle`   | Угол поворота левого уха, в градусах                        |
| `right_ear_angle` | Угол поворота правого уха, в градусах                       |

**Коды ответа:**
| Код  | Значение                                                                                  |
|------|-------------------------------------------------------------------------------------------|
| `0`  | Успешное выполнение                                                                       |
| `-1` | Угол левого уха вне диапазона (см. `robohead_control/config/ears_driver.yaml`)                           |
| `-2` | Угол правого уха вне диапазона (см. `robohead_control/config/ears_driver.yaml`)                         |

:::note
- Диапазон углов: от -90° до +90° для каждого уха.
- **Положительное значение**: движение вперёд
    - *Левое ухо*: против часовой стрелки
    - *Правое ухо*: по часовой стрелке
- **Отрицательное значение**: движение назад
    - *Левое ухо*: по часовой стрелке
    - *Правое ухо*: против часовой стрелки
:::

*Подробнее о настройке сервоприводов:* [->](../../30-device-configuration/30-device-setting/34-configuring-servos.md)

## Режим отладки

В режиме отладки пакет `ears_driver` запускается изолированно (отдельно) для тестирования функций, без участия других компонентов системы.

### Шаг 1. Остановка всех запущенных пакетов

Остановите фоновый Linux-сервис:

```bash
sudo systemctl stop robohead.service
```

---

### Шаг 2. Запуск пакета вручную
Запустите пакет отдельно через launch-файл:
```bash
roslaunch ears_driver ears_driver_py.launch
```

---

### Шаг 3. Особенности работы в режиме отладки
- **Пространство имен**: сервис пакета **не имеет приставки** `/robohead_controller/`. Используется `/ears_driver/EarsSetAngle` вместо `/robohead_controller/ears_driver/EarsSetAngle`

- **Файл конфигурации**: настройки берутся из `ears_driver/config/ears_driver.yaml` вместо `robohead_controller/config/ears_driver.yaml`

---

### Шаг 4. Возможности тестирования

**ROS-сервис `EarsSetAngle`** позволяет задать позицию ушных сервоприводов.

   **Пример вызова:**

   ```bash
   # Запустите в отдельном терминале
rosservice call /ears_driver/EarsSetAngle "left_ear_angle: -30
right_ear_angle: 30"
   ```
Убедитесь, что вы находитесь в режиме отладки!

**Пример использования сервиса в Python:**

Команда для запуска примера с использованием сервиса `/ears_driver/EarsSetAngle` в Python:

```bash
# Запустите в отдельном терминале
rosrun ears_driver example_zigzag.py
```
Убедитесь, что вы находитесь в режиме отладки!