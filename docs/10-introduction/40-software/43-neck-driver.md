---
id: 43-neck-driver
slug: neck_driver
title: "Пакет neck_driver"
sidebar_label: "neck_driver"
description: "neck_driver"
draft: true
---

# Пакет `neck_driver` 

**Назначение:** Пакет `neck_driver` позволяет взаимодействовать с шейным суставом головы: плавно перемещать голову по двум углам (вертикали и горизонтали).

---

## Содержание пакета

```text
neck_driver/
├── CMakeLists.txt
├── package.xml
├── launch/
│   └── neck_driver_py.launch
├── config/
│   └── neck_driver.yaml
├── scripts/
│   └── main.py
├── srv/
│   └── NeckSetAngle.srv
└── examples/
    └── example_zigzag.py
```

---

## Запуск пакета

- Пакет `neck_driver` запускается автоматически при старте устройства.
- Запуск инициируется через launch-файл `robohead_controller_py.launch` из пакета `robohead_controller`.

---

## Основные возможности

Пакет `neck_driver` предоставляет **ROS-сервис `NeckSetAngle`** для управления шейным суставом головы:

**ROS-сервис `NeckSetAngle`** позволяет задать позицию шейного сустава с контролем углов и времени движения.

   **Пример вызова:**

   ```bash
rosservice call robohead_controller/neck_driver/NeckSetAngle "vertical_angle: 15
horizontal_angle: 0
duration: 1.5
is_blocking: 1"
   ```



**Параметры:**
| Параметр           | Описание                                                                 |
|--------------------|--------------------------------------------------------------------------|
| `vertical_angle`   | Угол тангажа (вертикальное движение), в градусах                         |
| `horizontal_angle` | Угол рыскания (горизонтальное движение), в градусах                       |
| `duration`         | Время перемещения до заданного положения, в секундах                      |
| `is_blocking`      | Блокирующий вызов: `1` — ждать окончания движения, `0` — не блокировать   |

**Коды ответа:**
| Код  | Значение                                                                                  |
|------|-------------------------------------------------------------------------------------------|
| `0`  | Успешное выполнение                                                                       |
| `-1` | Угол по вертикали вне диапазона (см. `robohead_control/config/neck_driver.yaml`)                            |
| `-2` | Угол по горизонтали вне диапазона (см. `robohead_control/config/neck_driver.yaml`)                          |
| `-3` | Негативное значение `duration` (должно быть ≥ 0)                                         |

:::note
• Рекомендуемый диапазон углов: **-30°…+30°** для обеих осей.
• Если указать `duration: 0`, перемещение будет максимально быстрым — использовать с осторожностью.
:::

*Подробнее о настройке сервоприводов:* [->](../../30-device-configuration/30-device-setting/34-configuring-servos.md)

## Режим отладки

В режиме отладки пакет `neck_driver` запускается изолированно (отдельно) для тестирования функций, без участия других компонентов системы.

### Шаг 1. Остановка всех запущенных пакетов

Остановите фоновый Linux-сервис:

```bash
sudo systemctl stop robohead.service
```

---

### Шаг 2. Запуск пакета вручную
Запустите пакет отдельно через launch-файл:
```bash
roslaunch neck_driver neck_driver_py.launch
```

---

### Шаг 3. Особенности работы в режиме отладки
- **Пространство имен**: сервис пакета **не имеет приставки** `/robohead_controller/`. Используется `/neck_driver/NeckSetAngle` вместо `/robohead_controller/neck_driver/NeckSetAngle`

- **Файл конфигурации**: настройки берутся из `neck_driver/config/neck_driver.yaml` вместо `robohead_controller/config/neck_driver.yaml`

---

### Шаг 4. Возможности тестирования

**ROS-сервис `NeckSetAngle`** позволяет задать позицию шейного сустава с контролем углов и времени движения.

   **Пример вызова:**

   ```bash
   # Запустите в отдельном терминале
   rosservice call neck_driver/NeckSetAngle "vertical_angle: 15
   horizontal_angle: 0
   duration: 1.5
   is_blocking: 1"
   ```
Убедитесь, что вы находитесь в режиме отладки!

**Пример использования сервиса в Python:**

Команда для запуска примера с использованием сервиса `/neck_driver/NeckSetAngle` в Python:

```bash
# Запустите в отдельном терминале
rosrun neck_driver example_zigzag.py
```
Убедитесь, что вы находитесь в режиме отладки!
