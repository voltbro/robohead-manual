---
id: 41-robohead-controller
slug: robohead_controller
title: Пакет robohead_controller
sidebar_label: "robohead_controller"
description: "robohead_controller"
draft: true
---

<!-- # Пакет `robohead_controller` -->

## Общее описание

Пакет `robohead_controller` — **центральный управляющий пакет** робоголовы. Он связывает все драйверы (уши, шея, дисплей, аудио, микрофон, камера, датчик батареи, распознавание речи) в единую систему с голосовым управлением и автоматическим поведением.

Контроллер реализует:
- **Подключение ко всем драйверам** через ROS2 сервисы и топики.
- **Голосовое управление** по схеме: wake phrase → распознавание команды → выполнение действия.
- **Систему действий (actions)** — модульные Python-скрипты, описывающие поведение робота.
- **Мониторинг батареи** с автоматической реакцией на низкий заряд.
- **Быстрые команды** — команды, выполняемые без wake phrase (громче, тише и т.д.).

---
## Структура пакета

```
robohead_controller
├── config                              # Конфигурационные файлы
│   ├── ears_driver.yaml
│   ├── media_driver.yaml
│   ├── neck_driver.yaml
│   ├── respeaker_driver.yaml
│   ├── robohead_controller.yaml        # Основной конфиг-файлы
│   ├── sensor_driver.yaml
│   ├── silero_tts.yaml
│   ├── speech_recognizer_asr.yaml
│   └── speech_recognizer_kws.yaml
├── launch
│   ├── dependencies.launch.py          # Launch-файл запуска зависимостей (все driver-пакеты)
│   └── robohead_controller.launch.py   # Launch-файл запуска пакета
├── package.xml
├── README.md
├── resource
│   └── robohead_controller
├── robohead_controller
│   ├── actions                         # Действия
│   │   ├── std_attention               # Стандартное действие "Внимание"
│   │   │   ├── action.py
│   │   │   ├── attention.gif
│   │   │   ├── attention.mp3
│   │   │   ├── attention.mp4
│   │   │   ├── attention.png
│   │   │   └── __pycache__
│   │   │       └── action.cpython-312.pyc
│   │   ├── std_ball_tracker
│   │   │   ├── action.py
│   │   │   ├── calibrate_voice.mp3
│   │   │   ├── finish_voice.mp3
│   │   │   └── __pycache__
│   │   │       └── action.cpython-312.pyc
│   │   ├── std_ears
│   │   ├── std_echo
│   │   ├── std_greeting
│   │   ├── std_left_ear
│   │   ├── std_llm
│   │   ├── std_low_bat
│   │   ├── std_make_photo
│   │   ├── std_right_ear
│   │   ├── std_show_voltage
│   │   ├── std_startup
│   │   ├── std_wait
│   │   ├── std_volume_down             # Быстрое действие на команду "Тише"
│   │   │   ├── action.py
│   │   │   └── set_vol.mp3
│   │   └── std_volume_up               # Быстрое действие на команду "Громче"
│   │       ├── action.py
│   │       ├── max_vol.mp3
│   │       └── set_vol.mp3
│   ├── controller.py                   # Файл с главным классом RoboheadController
│   ├── core
│   │   ├── action_manager.py           # Управление запуском действий
│   │   ├── battery_monitor.py          # Мониторинг состояния АКБ
│   │   ├── commander.py                # Переключение состояний: Ожидание ключевой фразы, распознавание команд
│   ├── drivers                         # Коннекторы для driver-пакетов
│   │   ├── ears_driver.py
│   │   ├── media_driver.py
│   │   ├── neck_driver.py
│   │   ├── respeaker_driver.py
│   │   ├── sensor_driver.py
│   │   ├── silero_tts.py
│   │   ├── speech_recognizer_asr.py
│   │   ├── speech_recognizer_kws.py
│   │   └── usb_cam.py
│   ├── __init__.py
│   ├── main.py                         # Точка входа
├── setup.cfg
├── setup.py
└── test
    ├── test_copyright.py
    ├── test_flake8.py
    └── test_pep257.py
```

---

## Автоматический запуск

* Пакет `robohead_controller` стартует автоматически при загрузке устройства.
> Для отключения автозапуска используйте: `sudo systemctl stop robohead.service`
* Запуск этого пакета инициирует старты всех зависимых ROS-пакетов (см. `launch/robohead_controller_py.launch` и `launch/dependencies.launch`).

---

## Action-скрипты

При распознавании стандартных голосовых команд вызывается соответствующий action-скрипт:

| Скрипт             | Описание команды                     |
| ------------------ | ------------------------------------ |
| `std_startup`      | Действие при запуске                 |
| `std_wait`         | Ожидание ключевой фразы              |
| `std_attention`    | «Слушай, Робот!»                     |
| `std_ball_tracker` | «Следи за шариком»                   |
| `std_ears`         | «Покажи уши»                         |
| `std_left_ear`     | «Покажи левое ухо»                   |
| `std_right_ear`    | «Покажи правое ухо»                  |
| `std_greeting`     | «Поздоровайся»                       |
| `std_make_photo`   | «Сделай фото»                        |
| `std_echo`         | "Повтори за мной"                    |
| `std_llm`          | "Ответь на вопрос"                   |
| `std_low_bat`      | Низкий заряд — блокирование других скриптов |

> *Расположение:* `robohead_controller/robohead_controller/actions/<script_name>/`

Каждый скрипт содержит код `action.py` и мультимедиа (картинки, аудио, видео). Для замены медиа без изменения кода просто замените файл в папке скрипта, сохранив имя и расширение.

---

## Конфигурация параметров устройства

Основные параметры для подключенных аппаратных компонентов находятся в папке `robohead_controller/config/`:

* `media_driver.yaml` — настройка параметров дисплея
* `ears_driver.yaml` — настройка параметров ушных сервоприводов
* `neck_driver.yaml` — настройка параметров шейных сервоприводов
* `respeaker_driver.yaml` — настройка параметров микрофонного массива
* `sensor_driver.yaml` — настройка параметров датчика АКБ
* `silro_tts.yaml` - настройка параметров tts (Text to Speech) пакета Silero
* `speech_recognizer_asr.yaml` - настройка параметров пакета speech_recognizer для свободного распознавания и распознавания по грамматике
* `speech_recognizer_kws.yaml` - настройка параметров пакета speech_recognizer для ключевых слов (wake_phrases) и быстрых команд (fast_commands)
* `robohead_controller.yaml` - главный конфигурационный файл пакета robohead_controller

Настройка соответствия "голосовая команда - action-скрипт" задается в файле `config/robohead_controller.yaml`.

## TODO проверить ссылку
Подробнее про настройку параметров написано здесь [->](../../30-device-configuration/30-device-setting/30-changing-device-settings.md)

---

## Основной цикл работы
![Состояния robohead_controller](../attachments/state_machine.png)

---

## Компоненты ядра

### `main.py` — Точка входа

Создаёт экземпляр `RoboheadController`, подключает все драйверы, запускает стартовое действие и `MultiThreadedExecutor` для параллельной обработки колбэков.

### `controller.py` — Главный класс

Класс `RoboheadController` (наследник `rclpy.Node`):

- Хранит экземпляры всех коннекторов драйверов.
- Хранит очереди событий: `queue_wake_phrases`, `queue_commands`, `queue_fast_commands`.
- Предоставляет метод `sleep(cancel_event, duration)` для использования в действиях (с поддержкой отмены).
- Загружает маппинг `actions_match` из конфига (имя команды → путь к Python-скрипту).
- Флаг `is_allow_work` управляется `BatteryMonitor`.

### `action_manager.py` — Менеджер действий

Управляет запуском, отменой и изоляцией действий:

- **`execute_action(name:str, on_complete: Optional[str], cancelling:bool)`** — запуск действия:
  - Если `cancelling=True` — отменяет текущее действие перед запуском нового (обычное действие).
  - Если `cancelling=False` — НЕ отменяет текущее действие перед запуском (быстрое действие).
  - Действие выполняется в **отдельном daemon-потоке**.
  - После успешного завершения может автоматически запустить действие `on_complete`.

- **Изоляция ошибок** — ошибки в действиях логируются, но не приводят к падению контроллера.
- **Отмена** — через `threading.Event`. Действие должно периодически проверять `cancel_event.is_set()`. Если `cancel_event.is_set()==True`, то действие должно завершиться.

### `battery_monitor.py` — Мониторинг батареи

Проверяет напряжение батареи каждые **0.5 секунды**:

| Условие | Действие |
|---------|----------|
| `voltage < low_voltage_threshold` и работа разрешена (`is_allow_work == True`) | Запрещает работу (`is_allow_work = False`), запускает `std_low_bat` |
| `voltage ≥ (threshold + hysteresis)` и работа запрещена (`is_allow_work == False`) | Разрешает работу (`is_allow_work = True`), запускает `std_wait` |

Гистерезис предотвращает частое переключение при напряжении вблизи порога.

### `commander.py` — Обработчик команд

Работает с частотой **10 Гц**, реализует конечный автомат:

#TODO сделать хорошую картинку конечного автомата
![Конечный автомат commander.py](../attachments/commander_scheme.png)


**Быстрые команды** обрабатываются **в любом состоянии** без прерывания текущего действия (`cancelling=False`).

---

## Коннекторы драйверов

Каждый коннектор — обёртка над ROS2 клиентами/подписчиками с поддержкой `cancel_event` и блокирующего/неблокирующего режима.

### `EarsDriverConnector`

| Метод | Описание |
|-------|----------|
| `connect()` | Подключение к сервисам `ears_set_angle`, `is_idle` |
| `set_angle(cancel_event, left, right, duration, block)` | Установка углов ушей. `block=True` — ждать завершения движения |
| `is_idle(cancel_event)` | Проверка завершения движения |

### `NeckDriverConnector`

| Метод | Описание |
|-------|----------|
| `connect()` | Подключение к сервисам `neck_set_angle`, `is_idle` |
| `set_angle(cancel_event, vertical, horizontal, duration, block)` | Установка углов шеи. `block=True` — ждать завершения |
| `is_idle(cancel_event)` | Проверка завершения движения |

### `MediaDriverConnector`

| Метод | Описание |
|-------|----------|
| `connect()` | Подключение ко всем сервисам и топикам media_driver |
| `play_media(cancel_event, video_path, audio_path, loop, block)` | Воспроизведение видео + аудио |
| `play_audio(cancel_event, audio_path, loop, block)` | Воспроизведение только аудио |
| `play_display(cancel_event, video_path, loop, block)` | Отображение только видео/изображения |
| `set_volume(cancel_event, volume)` | Установка громкости (0–100) |
| `get_volume(cancel_event)` | Получение текущей громкости |
| `is_idle_audio(cancel_event)` | Проверка завершения аудио |
| `is_idle_display(cancel_event)` | Проверка завершения видео |
| `stream_publish(image_msg)` | Публикация кадра в стрим |

### `SensorDriverConnector`

| Метод/Поле | Описание |
|------------|----------|
| `connect()` | Подписка на топик `battery` |
| `battery_voltage` | Текущее напряжение (В) |
| `battery_current` | Текущий ток (А) |
| `battery_power_supply_status` | Статус питания |
| `battery_power_supply_technology` | Тип АКБ (Li-Ion) |
| `battery_state` | "Сырое" сообщение BatteryState |



### `RespeakerDriverConnector`

| Метод/Поле | Описание |
|------------|----------|
| `connect()` | Подключение к сервисам LED и подписка на DOA, аудио |
| `doa` | Текущий угол DOA (градусы) |
| `audio_main` | Последний аудиофрейм основного канала |
| `set_led_mode(cancel_event, mode)` | Устанавливает режим работы LED-кольца |
| `set_led_brightness(cancel_event, value)` | Устанавливает яркость LED-кольца |
| `set_led_color_all(cancel_event, red, green, blue)` | Устанавливает одинаковый цвет всем светодиодам LED-кольца |
| `set_led_color_palette(cancel_event, red_a, green_a, blue_a, red_b, green_b, blue_b)` | Устанавливает двухцветную палитру для анимационных режимов LED |
| `set_led_color_manual(cancel_event, colors)` | Устанавливает каждому (из 12) светодиоду LED-кольца отдельный цвет |



### `SpeechRecognizerAsrConnector`

| Метод | Описание |
|-------|----------|
| `connect()` | Подключение к сервису `set_mode` и подписка на `commands`, `frees` |
| `set_mode(mode)` | Установка режима ASR (0/1/2) |
| Колбэк `commands` | Полученные команды добавляются в `controller.queue_commands` |
| Колбэк `frees` | Полученные команды добавляются в `controller.queue_frees` |


### `SpeechRecognizerKwsConnector`

| Метод | Описание |
|-------|----------|
| `connect()` | Подключение к сервису `set_mode` и подписка на `wake_phrases`, `fast_commands` |
| `set_mode(mode)` | Установка режима KWS (0/1) |
| Колбэк `wake_phrases` | Полученные фразы добавляются в `controller.queue_wake_phrases` |
| Колбэк `fast_commands` | Полученные команды добавляются в `controller.queue_fast_commands` |

### `UsbCamConnector`

| Метод/Поле | Описание |
|------------|----------|
| `connect()` | Подписка на топик `image_raw` |
| `image_raw` | Последний кадр с камеры (`sensor_msgs/msg/Image`) |

---

## Система действий (Actions)

### Структура действия

Каждое действие — Python-файл с функцией `run()`:

```python
def run(controller, action_name: str, cancel_event: threading.Event):
    """
    Args:
        controller: Ссылка на RoboheadController (доступ ко всем драйверам)
        action_name: Имя действия (строка). Совпадает с командой, которой был вызван этот action.
        cancel_event: threading.Event для проверки отмены
    """
    # Логика действия...
```

### Правила написания действий

1. **Периодически проверять `cancel_event.is_set()`** — для корректной отмены. Методы коннекторов драйверов делают внутри себя это за вас. Обращайте на это внимание при написании длительных операций (длительные циклы, time.sleep (не рекомендуется) и т.д.).
2. **Передавать `cancel_event`** во все вызовы драйверов.
3. **Не использовать `time.sleep()` напрямую** — использовать `controller.sleep(cancel_event, duration)` или собственную обёртку с проверкой отмены.
4. Ошибки в действиях **изолированы** — не приводят к падению контроллера.

### Стандартные действия

| Имя | Файл | Описание |
|-----|------|----------|
| `std_startup` | `actions/std_startup/action.py` | Стартовое приветствие: воспроизведение видео и аудио при запуске |
| `std_wait` | `actions/std_wait/action.py` | Режим ожидания: зацикленная анимация на дисплее, остановка аудио, уши и шея в нейтральное положение |
| `std_attention` | `actions/std_attention/action.py` | Реакция на wake phrase: анимация внимания, поворот головы к источнику звука (DOA), движение ушей |
| `std_low_bat` | `actions/std_low_bat/action.py` | Реакция на низкий заряд батареи |
| `std_ears` | `actions/std_ears/action.py` | Демонстрация ушей: анимация с поочерёдным движением ушей и покачиванием головы |
| `std_greeting` | `actions/std_greeting/action.py` | Приветствие по голосовой команде |
| `std_ball_tracker` | `actions/std_ball_tracker/action.py` | Слежение за шариком с камеры |
| `std_make_photo` | `actions/std_make_photo/action.py` | Фотографирование с камеры |
| `std_left_ear` | `actions/std_left_ear/action.py` | Демонстрация левого уха |
| `std_right_ear` | `actions/std_right_ear/action.py` | Демонстрация правого уха |
| `std_show_voltage` | `actions/std_show_voltage/action.py` | Показывает текущие напряжение и ток на АКБ |
| `std_echo` | `actions/std_echo/action.py` | Записывает вашу речь и повторяет своим голосом |
| `std_llm` | `actions/std_llm/action.py` | Распознает ваш вопрос и отправляет большой языковой модели, полученный от неё ответ озвучивается

### Быстрые действия (Fast Actions)

Выполняются **без прерывания** текущего действия (`cancelling=False`). Используются для команд, которые должны сработать мгновенно. Для их вызова обращаться к роботу (**"Слушай, Робот!"**) не нужно!

| Имя | Файл | Описание |
|-----|------|----------|
| `громче` | `fast_actions/std_volume_up/action.py` | Увеличение громкости на 10%. При достижении максимума — звуковое уведомление |
| `тише` | `fast_actions/std_volume_down/action.py` | Уменьшение громкости на 10% |

### Маппинг действий (`actions_match`)

Соответствие между именем команды и Python-скриптом задаётся в конфиге как JSON:

```yaml
actions_match: >
  {
    "std_startup":        "std_startup/action.py",
    "std_wait":           "std_wait/action.py",
    "поздоровайся":       "std_greeting/action.py",
    "покажи уши":         "std_ears/action.py",
    "громче":             "std_volume_up/action.py"
  }
```

- **Относительные пути** разрешаются от `robohead_controller/actions/`.
- **Абсолютные пути** используются как есть.
- **Ключи** должны совпадать с командами в `speech_recognizer_asr.yaml` (`commands`) и `speech_recognizer_kws.yaml` (`fast_commands`).

---

## Параметры конфигурации

Файл: `config/robohead_controller.yaml`

### Основные параметры

| Параметр | Тип | По умолчанию | Описание |
|----------|-----|-------------|----------|
| `low_voltage_threshold` | `float` | `6.3` | Порог низкого напряжения батареи (В). Ниже — запуск `std_low_bat` |
| `low_voltage_hysteresis` | `float` | `0.5` | Гистерезис восстановления (В). Работа разрешается при `voltage ≥ threshold + hysteresis` |
| `wait_timeout` | `float` | `1.0` | Таймаут ожидания доступности сервисов при подключении (сек) |
| `actions_match` | `string` (JSON) | `"{}"` | JSON-маппинг: имя команды → путь к скрипту действия |

### Имена сервисов и топиков драйверов

Конфиг содержит полные имена (с namespace) всех сервисов и топиков для подключения к драйверам:

```yaml
media_driver:
  srv_play_media_name: "/robohead/media_driver/play_media"
  # ...

ears_driver:
  srv_ears_set_angle_name: "/robohead/ears_driver/ears_set_angle"
  # ...

neck_driver:
  srv_neck_set_angle_name: "/robohead/neck_driver/neck_set_angle"
  # ...

respeaker_driver:
  topic_name:
    doa: "/robohead/respeaker_driver/doa"
    # ...
  service_name:
    set_mode: "/robohead/respeaker_driver/set_mode"
    # ...

sensor_driver:
  topic_name: "/robohead/sensor_driver/battery"

speech_recognizer_asr:
  service_name:
    set_mode: "/robohead/speech_recognizer/asr/set_mode"
  topic_name:
    commands: "/robohead/speech_recognizer/asr/commands"

speech_recognizer_kws:
  service_name:
    set_mode: "/robohead/speech_recognizer/kws/set_mode"
  topic_name:
    wake_phrases: "/robohead/speech_recognizer/kws/wake_phrases"
    fast_commands: "/robohead/speech_recognizer/kws/fast_commands"

usb_cam:
  topic_name_image_raw: "/robohead/usb_cam/image_raw"
```

---

## Launch-файлы

### `robohead_controller.launch.py` — Полный запуск

Запускает **все драйверы** и **контроллер**:

```bash
ros2 launch robohead_controller robohead_controller.launch.py
```
> Эта же команда запускается из скрипта ~/start.sh, который вызывается автоматически при запуске Rasperry Pi сервисом robohead.service

Последовательность:
1. `dependencies.launch.py` — запуск всех драйверов.
2. `robohead_controller` — запуск контроллера.

### `dependencies.launch.py` — Только драйверы

Запускает все драйверы без контроллера (для отладки):

```bash
ros2 launch robohead_controller dependencies.launch.py
```

Запускаемые ноды (namespace `/robohead/`):

| Нода | Пакет | Namespace |
|------|-------|-----------|
| `media_driver_node` | `media_driver` | `/robohead/media_driver/` |
| `neck_driver` | `neck_driver` | `/robohead/neck_driver/` |
| `sensor_driver` | `sensor_driver` | `/robohead/sensor_driver/` |
| `ears_driver` | `ears_driver` | `/robohead/ears_driver/` |
| `usb_cam` | `usb_cam` | `/robohead/usb_cam/` |
| `respeaker_driver` | `respeaker_driver` | `/robohead/respeaker_driver/` |
| `speech_recognizer_kws_node` | `speech_recognizer` | `/robohead/speech_recognizer/kws/` |
| `speech_recognizer_asr_node` | `speech_recognizer` | `/robohead/speech_recognizer/asr/` |

---

## Пример создания нового действия
1. Создать папку действия в `robohead_controller/robohead_controller/actions/` и скрипт `action.py`
2. Добавить в маппинг (`robohead_controller.yaml`)
3. Добавить команду в ASR (`speech_recognizer_asr.yaml`)
4. Пересобрать и запустить
```bash
colcon build --symlink-install --packages-select robohead_controller speech_recognizer
ros2 launch robohead_controller robohead_controller.launch.py
```

---

## Сборка

```bash
colcon build --symlink-install --packages-select robohead_interfaces robohead_controller
```

## Запуск

```bash
# Полный запуск (драйверы + контроллер)
ros2 launch robohead_controller robohead_controller.launch.py

# Только драйверы (для отладки)
ros2 launch robohead_controller dependencies.launch.py
```