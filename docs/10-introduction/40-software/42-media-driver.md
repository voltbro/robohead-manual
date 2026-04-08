---
id: 42-media-driver
slug: media_driver
title: "Пакет media_driver"
sidebar_label: "media_driver"
description: "media_driver"
draft: true
---
# Документация: ROS2 пакет `media_driver`

## Общее описание

Пакет `media_driver` обеспечивает управление мультимедийным контентом на робоголове через **два независимых экземпляра MPV-плеера**:

- **Video player** — отображает видео/изображения на дисплее через DRM (без X-сервера), аудиодорожка отключена
- **Audio player** — воспроизводит аудиофайлы через ALSA, видеовыход отключен

Такая архитектура позволяет **одновременно** показывать визуальный контент на экране и воспроизводить звук. Дополнительно поддерживается потоковый вывод ROS-изображений на дисплей в реальном времени.

---

## Структура пакета

```
media_driver/
├── CMakeLists.txt
├── config
│   └── media_driver.yaml
├── examples
│   ├── audio.mp3
│   ├── picture.png
│   └── video.mp4
├── include
│   └── media_driver
│       ├── mpv_player.hpp
│       ├── node.hpp
│       └── utils.hpp
├── launch
│   └── media_driver.launch.py
├── package.xml
├── README.md
└── src
    ├── imager.py
    ├── main.cpp
    ├── mpv_player.cpp
    ├── node.cpp
    └── utils.cpp
```

---

## Зависимости

### Системные библиотеки
- **libmpv** — медиаплеер
- **OpenCV** — обработка изображений
- **ALSA** — аудиовыход

### ROS2 пакеты
- `rclcpp` — клиентская библиотека C++
- `sensor_msgs` — стандартные сообщения для изображений
- `cv_bridge` — конвертация ROS ↔ OpenCV
- `robohead_interfaces` — пользовательские сервисы (`PlayMedia`, `SimpleCommand`)

---

## Сборка и запуск

### Сборка

```bash
cd ~/robohead_ws
colcon build --symlink-install --packages-select robohead_interfaces media_driver
source install/setup.bash
```

### Запуск

```bash
ros2 launch media_driver media_driver.launch.py
```

**Ожидаемый вывод:**
```
[INFO] [media_driver_node-1]: process started with pid [15077]
[INFO] [media_driver]: [mpv_video] MPV player initialized successfully (type: VIDEO)
[INFO] [media_driver]: [audio_mpv] MPV player initialized successfully (type: AUDIO)
[INFO] [media_driver]: INITED
```

---

## API: Сервисы

### 1. `play_media` — Воспроизведение медиа

**Тип:** `robohead_interfaces/srv/PlayMedia`  
**Полное имя:** `/media_driver/play_media`

#### Поля запроса

| Поле | Тип | Описание |
|------|-----|----------|
| `path_to_video_file` | `string` | Путь к видео/изображению, URL потока или `__STOP__` для остановки.<br/>Пустая строка — не трогать видеоплеер |
| `path_to_audio_file` | `string` | Путь к аудиофайлу, URL потока или `__STOP__` для остановки.<br/>Пустая строка — не трогать аудиоплеер |
| `loop` | `bool` | `true` — зацикленное воспроизведение, `false` — однократное |

#### Поля ответа

| Поле | Тип | Значение |
|------|-----|----------|
| `data` | `int16` | `0` — успех<br/>`-1` — ошибка (файл не найден, неподдерживаемый формат) |

#### Поддерживаемые форматы

| Категория | Расширения |
|-----------|-----------|
| **Изображения** | `.png`, `.jpg`, `.jpeg`, `.bmp`, `.webp`, `.gif`, `.tiff`, `.svg`, `.heic`, `.heif` |
| **Видео** | `.mp4`, `.mov`, `.avi`, `.mkv`, `.webm`, `.flv`, `.wmv`, `.mpeg`, `.3gp`, `.ts` |
| **Аудио** | `.mp3`, `.wav`, `.ogg`, `.flac`, `.aac`, `.m4a`, `.wma`, `.opus`, `.aiff`, `.midi` |

#### Поддерживаемые протоколы

`http://`, `https://`, `rtsp://`, `rtmp://`, `mms://`, `hls://`, `dash://`

#### Примеры

```bash
# Воспроизвести аудио
ros2 service call /media_driver/play_media robohead_interfaces/srv/PlayMedia \
  "{path_to_video_file: '', path_to_audio_file: '/path/to/audio.mp3', loop: false}"

# Показать видео (зацикленное)
ros2 service call /media_driver/play_media robohead_interfaces/srv/PlayMedia \
  "{path_to_video_file: '/path/to/video.mp4', path_to_audio_file: '', loop: true}"

# Одновременно: видео + аудио
ros2 service call /media_driver/play_media robohead_interfaces/srv/PlayMedia \
  "{path_to_video_file: '/path/to/video.mp4', path_to_audio_file: '/path/to/audio.mp3', loop: false}"

# Воспроизвести интернет-радио
ros2 service call /media_driver/play_media robohead_interfaces/srv/PlayMedia \
  "{path_to_video_file: '', path_to_audio_file: 'http://example.com/stream.mp3', loop: false}"

# Остановить видео (аудио продолжит играть)
ros2 service call /media_driver/play_media robohead_interfaces/srv/PlayMedia \
  "{path_to_video_file: '__STOP__', path_to_audio_file: '', loop: false}"

# Остановить всё
ros2 service call /media_driver/play_media robohead_interfaces/srv/PlayMedia \
  "{path_to_video_file: '__STOP__', path_to_audio_file: '__STOP__', loop: false}"
```

---

### 2. `set_volume` — Установка громкости

**Тип:** `robohead_interfaces/srv/SimpleCommand`  
**Полное имя:** `/media_driver/set_volume`

#### Поля запроса

| Поле | Тип | Описание |
|------|-----|----------|
| `data` | `int16` | Желаемая громкость (0–100) |

#### Поля ответа

| Поле | Тип | Значение |
|------|-----|----------|
| `data` | `int16` | Установленная громкость или `-1` при ошибке |

#### Пример

```bash
ros2 service call /media_driver/set_volume robohead_interfaces/srv/SimpleCommand "{data: 75}"
```

---

### 3. `get_volume` — Получение громкости

**Тип:** `robohead_interfaces/srv/SimpleCommand`  
**Полное имя:** `/media_driver/get_volume`

#### Поля ответа

| Поле | Тип | Значение |
|------|-----|----------|
| `data` | `int16` | Текущая громкость (0–100) или `-1` при ошибке |

#### Пример

```bash
ros2 service call /media_driver/get_volume robohead_interfaces/srv/SimpleCommand "{data: 0}"
```

---

### 4. `is_idle/audio` — Проверка состояния аудиоплеера

**Тип:** `robohead_interfaces/srv/SimpleCommand`  
**Полное имя:** `/media_driver/is_idle/audio`

#### Поля ответа

| Поле | Тип | Значение |
|------|-----|----------|
| `data` | `int16` | `1` — плеер простаивает (idle, пауза, EOF)<br/>`0` — активное воспроизведение |

#### Логика определения idle

Плеер считается idle, если выполняется **любое** условие:
- `idle-active = yes` (ничего не загружено)
- `pause = yes` (на паузе)
- `eof-reached = yes` (конец файла)
- `path` пуст (файл не загружен)

#### Пример

```bash
ros2 service call /media_driver/is_idle/audio robohead_interfaces/srv/SimpleCommand "{data: 0}"
```

---

### 5. `is_idle/display` — Проверка состояния видеоплеера

**Тип:** `robohead_interfaces/srv/SimpleCommand`  
**Полное имя:** `/media_driver/is_idle/display`

#### Поля ответа

| Поле | Тип | Значение |
|------|-----|----------|
| `data` | `int16` | `1` — дисплей простаивает<br/>`0` — активное отображение |

#### Пример

```bash
ros2 service call /media_driver/is_idle/display robohead_interfaces/srv/SimpleCommand "{data: 0}"
```

---

## API: Топики

### Подписка: `stream` — Потоковый вывод изображений

**Тип:** `sensor_msgs/msg/Image`  
**Полное имя:** `/media_driver/stream`  
**QoS:** глубина очереди 1, `best_effort`

#### Описание

Принимает ROS-изображения (формат `bgr8`) и отображает их на дисплее в реальном времени. Используется для вывода динамического контента (анимации лица, интерфейса).

#### Механизм работы

1. Входящее изображение конвертируется в OpenCV `Mat`
2. Кадр сохраняется в `/dev/shm/robohead_stream_frame.ppm` (RAM, без диска)
3. Видеоплеер обновляет отображение через `loadfile ... replace`

> **Важно:** Стриминг не влияет на аудиоплеер и не требует остановки текущего воспроизведения

---

## Параметры конфигурации

Файл: `config/media_driver.yaml`

### Имена сервисов

| Параметр | Тип | По умолчанию | Описание |
|----------|-----|-------------|----------|
| `srv_play_media_name` | `string` | `play_media` | Имя сервиса воспроизведения |
| `srv_set_volume_name` | `string` | `set_volume` | Имя сервиса установки громкости |
| `srv_get_volume_name` | `string` | `get_volume` | Имя сервиса получения громкости |
| `srv_is_idle_audio_name` | `string` | `is_idle/audio` | Имя сервиса проверки состояния аудио |
| `srv_is_idle_display_name` | `string` | `is_idle/display` | Имя сервиса проверки состояния дисплея |

### Имена топиков

| Параметр | Тип | По умолчанию | Описание |
|----------|-----|-------------|----------|
| `topic_stream_name` | `string` | `stream` | Имя топика для приёма видеопотока |

### Настройки медиа

| Параметр | Тип | По умолчанию | Описание |
|----------|-----|-------------|----------|
| `display_rotate` | `string` | `270` | Угол поворота изображения (0, 90, 180, 270) |
| `default_volume` | `double` | `50.0` | Начальная громкость аудиоплеера (0–100) |
| `stop_command` | `string` | `__STOP__` | Команда для остановки плеера |

---

### Ключевые особенности

- **Независимость плееров**: видео и аудио работают параллельно
- **DRM-вывод**: прямой рендеринг на дисплей без оконного менеджера
- **Аппаратное ускорение**: `hwdec=auto` для декодирования видео
- **Статические изображения**: отображаются до 24 часов (`image-display-duration`)
- **Потоковая передача**: поддержка интернет-радио и RTSP-камер
- **Память вместо диска**: временные файлы в `/dev/shm`

---

## Примеры использования

### Базовые сценарии

```bash
# Показать картинку
ros2 service call /media_driver/play_media robohead_interfaces/srv/PlayMedia \
  "{path_to_video_file: '/home/pi/robohead_ws/src/robohead2/media_driver/examples/picture.png', path_to_audio_file: '', loop: false}"

# Воспроизвести видео с зацикливанием
ros2 service call /media_driver/play_media robohead_interfaces/srv/PlayMedia \
  "{path_to_video_file: '/home/pi/robohead_ws/src/robohead2/media_driver/examples/video.mp4', path_to_audio_file: '', loop: true}"

# Включить фоновую музыку
ros2 service call /media_driver/play_media robohead_interfaces/srv/PlayMedia \
  "{path_to_video_file: '', path_to_audio_file: '/home/pi/robohead_ws/src/robohead2/media_driver/examples/audio.mp3', loop: true}"
```

### Продвинутые сценарии
## TODO (прописать пути до example)

```bash
# Видео на экране + отдельное аудио
ros2 service call /media_driver/play_media robohead_interfaces/srv/PlayMedia \
  "{path_to_video_file: '/path/to/video.mp4', path_to_audio_file: '/path/to/music.mp3', loop: false}"

# Интернет-радио + статическое изображение
ros2 service call /media_driver/play_media robohead_interfaces/srv/PlayMedia \
  "{path_to_video_file: '/path/to/logo.png', path_to_audio_file: 'http://chanson.hostingradio.ru:8041/chanson256.mp3', loop: false}"

# Сменить только аудио (видео продолжает играть)
ros2 service call /media_driver/play_media robohead_interfaces/srv/PlayMedia \
  "{path_to_video_file: '', path_to_audio_file: '/path/to/new_audio.mp3', loop: false}"

# Остановить только видео
ros2 service call /media_driver/play_media robohead_interfaces/srv/PlayMedia \
  "{path_to_video_file: '__STOP__', path_to_audio_file: '', loop: false}"
```

---

## Дополнительные ресурсы

- [Документация libmpv](https://mpv.io/manual/master/)
- [ROS2 сервисы](https://docs.ros.org/en/jazzy/Tutorials/Services/Understanding-ROS2-Services.html)
- [sensor_msgs/Image](https://docs.ros2.org/latest/api/sensor_msgs/msg/Image.html)