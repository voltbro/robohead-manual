---
id: 45-speakers-driver
slug: speakers_driver
title: "Пакет speakers_driver"
sidebar_label: "speakers_driver"
description: "speakers_driver"
draft: true
---

# Пакет `speakers_driver`

**Назначение:** Пакет `speakers_driver` обеспечивает вывод звука на внешние динамики: воспроизведение аудио, установку громкости.

---

## Содержание пакета

```text
speakers_driver/
├── CMakeLists.txt
├── package.xml
├── launch/
│   └── speakers_driver.launch
├── config/
│   └── speakers_driver.yaml
├── scripts/
│   └── main.py
├── srv/
│   └── GetVolume.srv
│   └── PlayAudio.srv
│   └── SetVolume.srv
└── examples/
    └── example_file.py
    └── file.mp3
```

---

## Запуск пакета

- Пакет `speakers_driver` запускается автоматически при старте устройства.
- Запуск инициируется через launch-файл `robohead_controller_py.launch` из пакета `robohead_controller`.

---

## Основные возможности

Пакет `speakers_driver` предоставляет следующие функции для работы с внешними динамиками:

1. **Воспроизведение аудио-файлов**

   * **Сервис**: `robohead_controller/speakers_driver/PlayAudio`
   * **Параметр:**

     | Аргумент       | Описание                                             |
     | -------------- | ---------------------------------------------------- |
     | `path_to_file` | Путь к MP3/WAV на устройстве. :::tip Вместо аудио-файла можно указать путь до любого стримингового аудио-потока в Интернете (например, онлайн-радио). Если оставить пустым, то произойдет принудительная остановка текущего воспроизводимого аудио.:::                 |
     | `is_blocking`  | `1` — дождаться окончания; `0` — неблокирующий вызов |
     | `is_cycled`    | `1` — зациклить воспроизведение; `0` — разовое воспроизведение |
   * **Пример:**

    ```bash
    rosservice call robohead_controller/speakers_driver/PlayAudio "path_to_file: '/home/pi/robohead_ws/src/robohead/speakers_driver/examples/file.mp3'
    is_blocking: 1
    is_cycled: 0"
    ```
   * **Коды возврата:**

     | Код  | Описание       |
     | ---- | -------------- |
     | `0`  | Успех          |
     | `-1` | Файл не найден |



2. **Установка уровня громкости**

   * **Сервис:** `robohead_controller/speakers_driver/SetVolume`
   * **Параметр:**

     | Аргумент | Описание                      |
     | -------- | ----------------------------- |
     | `volume` | Громкость в процентах (0–100) |
   * **Пример:**

     ```bash
     rosservice call robohead_controller/speakers_driver/SetVolume "volume: 70"
     ```
   * **Коды возврата:**

     | Код  | Описание                     |
     | ---- | ---------------------------- |
     | `0`  | Успех                        |
     | `-1` | Значение вне диапазона 0–100 |

3. **Получение текущего уровня громкости**

   * **Сервис:** `robohead_controller/speakers_driver/GetVolume`
   * **Описание:** не требует аргументов
   * **Пример:**

     ```bash
     rosservice call robohead_controller/speakers_driver/GetVolume "{}"
     ```
   * **Возвращаемое значение:** текущее значение громкости (0–100 %)

*Подробнее о настройке громкости динамиков:* [->](../../30-device-configuration/30-device-setting/33-adjusting-volume.md)

## Режим отладки

В режиме отладки пакет `speakers_driver` запускается изолированно (отдельно) для тестирования функций, без участия других компонентов системы.

### Шаг 1. Остановка всех запущенных пакетов

Остановите фоновый Linux-сервис:

```bash
sudo systemctl stop robohead.service
```

---

### Шаг 2. Запуск пакета вручную
Запустите пакет отдельно через launch-файл:
```bash
roslaunch speakers_driver speakers_driver.launch 
```

---

### Шаг 3. Особенности работы в режиме отладки
- **Пространство имен**: сервисы пакета **не имеют приставки** `/robohead_controller/`. Используется `/speakers_driver/...` вместо `/robohead_controller/speakers_driver/...`

- **Файл конфигурации**: настройки берутся из `speakers_driver/config/speakers_driver.yaml` вместо `robohead_controller/config/speakers_driver.yaml`

---

### Шаг 4. Возможности тестирования

#### Воспроизведение аудиофайлов

**Форматы**: `.mp3`, `.wav`, `путь до любого стримингового аудио-потока в Интернете (например, онлайн-радио)`.

**Сервис**: `speakers_driver/PlayAudio`

**Параметры**:
  - `path_to_file` — путь к файлу (оставьте пустым для остановки аудио).
  - `is_blocking` — блокирующий вызов (`0` — нет, `1` — да). При блокирующем вызове сервиса в терминале, он будет висеть, пока воспроизведение не завершится. 
  - `is_cycled` — зацикленное воспроизведение (`0` — нет, `1` — да).

**Пример вызова сервиса:**

Воспроизведение файла находящегося по пути `~/robohead_ws/src/robohead/speakers_driver/examples/file.mp3` 

```bash
# Запустите в отдельном терминале
rosservice call /speakers_driver/PlayAudio "path_to_file: '/home/pi/robohead_ws/src/robohead/speakers_driver/examples/file.mp3'
is_blocking: 1
is_cycled: 0" 
```

#### Установка уровня громкости

**Сервис:** `speakers_driver/SetVolume`

**Параметр:**`volume` — громкость в процентах (0–100)

**Пример вызова сервиса:**

Пример установки громкости 30%

```bash
rosservice call /speakers_driver/SetVolume "volume: 30"
```

---

#### Получение текущего уровня громкости

**Сервис:** `/speakers_driver/GetVolume`

**Описание:** не требует аргументов
**Пример вызова сервиса:**

```bash
rosservice call /speakers_driver/GetVolume "{}" 
```
**Возвращаемое значение:** текущее значение громкости (0–100 %)

#### Пример работы с пакетом на Python

Команда для запуска примера с использованием пакета `speakers_driver` в Python:

```bash
# Запустите в отдельном терминале
rosrun speakers_driver example_file.py
```