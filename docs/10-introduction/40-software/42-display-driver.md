---
id: 42-display-driver
slug: display_driver
title: "Пакет display_driver"
sidebar_label: "display_driver"
description: "display_driver"
draft: true
---

# Пакет `display_driver`

**Назначение:** Пакет `display_driver` позволяет взаимодействовать с физическим экраном головы робота: выводить на него изображения и видео, выполнять потоковый вывод "сырых" изображений и обрабатывать данные с тачскрина.

---

## Содержание пакета

```
display_driver/
├── CMakeLists.txt
├── package.xml
├── launch/
│ └── display_driver_py.launch # Launch-файл для запуска пакета
├── config/
│ └── display_driver.yaml # Конфигурационный файл
├── scripts/
│ └── main.py # Основной исполняемый скрипт
├── srv/
│ └── PlayMedia.srv # Определение сервиса воспроизведения медиа
├── examples/
│ ├── example_file.py # Пример вывода статических файлов
│ ├── example_stream.py # Пример работы с видеопотоком
│ ├── example_touch.py # Пример обработки тачскрина
│ ├── pic.png # Тестовое изображение
│ └── vid.mp4 # Тестовое видео

```
---

## Запуск пакета

- Пакет `display_driver` запускается автоматически при старте устройства.
- Запуск инициируется через launch-файл `robohead_controller_py.launch` из пакета `robohead_controller`.

---

## Основные возможности

### 1. Вывод медиафайлов на экран

- **Поддерживаемые форматы**:
  - Картинки и анимации: `.jpg`, `.png`
  - Видео: `.mp4` (учтите, пакет воспроизводит только видео на экране. Для воспроизведения звука из видео используйте `speakers_driver`)

   :::warning
   Все медиа-материалы должны иметь разрешение **1080×1080** пикселей.
   :::

- Используется сервис ROS: `/robohead_controller/display_driver/PlayMedia`

---

### 2. Потоковая передача изображения

- Вывод произвольных изображений без задержек и проверок
- Используется ROS-топик `/robohead_controller/display_driver/PlayMedia` 

---

### 3. Обработка тачскрина

- Определение координат касаний
- Публикация в топик ROS: `/robohead_controller/display_driver/touchscreen`

---


## Режим отладки

В режиме отладки пакет `display_driver` запускается изолированно (отдельно) для тестирования функций, без участия других компонентов системы.

### Шаг 1. Остановка всех запущенных пакетов

Остановите фоновый Linux-сервис:

```bash
sudo systemctl stop robohead.service
```

---

### Шаг 2. Запуск пакета вручную
Запустите пакет отдельно через launch-файл:
```bash
roslaunch display_driver display_driver_py.launch
```

---

### Шаг 3. Особенности работы в режиме отладки
- **Пространство имен**: топики и сервисы пакета **не имеют приставки** `/robohead_controller/`. Используется `/display_driver/...` вместо `/robohead_controller/display_driver/...`

- **Файл конфигурации**: настройки берутся из `display_driver/config/display_driver.yaml` вместо `robohead_controller/config/display_driver.yaml`

---

### Шаг 4. Возможности тестирования

#### Воспроизведение медиафайлов

**Форматы**: `.jpg`, `.png`, `.mp4`.

**Сервис**: `/display_driver/PlayMedia`

**Параметры**:
  - `path_to_file` — путь к файлу (оставьте пустым для остановки медиа и очистки экрана).
  - `is_blocking` — блокирующий вызов (`0` — нет, `1` — да). При блокирующем вызове сервиса в терминале, он будет висеть, пока воспроизведение не завершится. 
  - `is_cycled` — зацикленное воспроизведение (`0` — нет, `1` — да).

   :::note
   При попытке вывода несуществующего файла вызов сервиса вернет -1. При несовпадении разрешения/расширения и других ошибках - подробная информация будет выведена на экран.
   :::

**Пример вызова сервиса:**

```bash
# Запустите в отдельном терминале
rosservice call /display_driver/PlayMedia "path_to_file: '/home/pi/robohead_ws/src/robohead/display_driver/examples/pic.png'
is_blocking: 0
is_cycled: 0"
```
Убедитесь, что вы находитесь в режиме отладки!
**Пример использования сервиса на Python:**

```bash
# Запустите в отдельном терминале
rosrun display_driver example_file.py
```
Убедитесь, что вы находитесь в режиме отладки!
#### Потоковая передача изображения

- **Топик**: `/display_driver/PlayMedia` (тип сообщения: `sensor_msgs/Image`)

:::note
Обратите внимание, что есть ROS-сервис `/display_driver/PlayMedia` и есть ROS-топик `/display_driver/PlayMedia`, для потокового вывода необходим именно **топик**.
:::

**Пример вывода изображения с веб-камеры:**
Запуск работы веб-камеры:
```bash
# Запустите в отдельном терминале пакет usb_cam
roslaunch usb_cam usb_cam.launch color_format:=yuv422p pixel_format:=mjpeg io_method:=mmap image_width:=640 image_height:=480 framerate:=25
```
Запуск скрипт-примера:
```bash
# Запустите в отдельном терминале скрипт-пример
rosrun display_driver example_stream.py
```

:::note
Убедитесь, что во время потокового вывода у вас не воспроизводятся медиа-материалы. Чтобы остановить воспроизведение вызовите сервис `/display_driver/PlayMedia` с пустым полем **path_to_file**
:::
Убедитесь, что вы находитесь в режиме отладки!

---

#### Тачскрин

- Топик: `/display_driver/touchscreen` (`geometry_msgs/Pose2D`)

**Пример просмотра данных:**

```bash
# Запустите в отдельном терминале
rostopic echo /display_driver/touchscreen
```

**Пример на Python:**

```bash
# Запустите в отдельном терминале
rosrun display_driver example_touch.py
```
Убедитесь, что вы находитесь в режиме отладки!
:::warning
Во время подключенного зарядного устройства могут наблюдаться шумы в данных с тачскрина!
:::