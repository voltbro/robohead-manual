---
id: 41-robohead-controller
slug: robohead_controller
title: "Пакет robohead_controller"
sidebar_label: "robohead_controller"
description: "robohead_controller"
draft: true
---

# Пакет `robohead_controller`

**Назначение:** основной ROS-пакет для управления всеми системами Робоголовы на основе распознанных аудиокоманд.

---

# Содержание пакета

```
robohead_controller/
├── CMakeLists.txt
├── package.xml
├── setup.py
├── launch/
│   └── dependencies.launch
│   └── robohead_controller_py.launch #основной launch-файл
├── config/
│   └── voice_recognizer_pocketsphinx/ #настройка голосовых команд и распознования
│       └── dictionary.dict
│       └── dictionary.dict.accent
│       └── dictionary.txt
│       └── gram.txt
│       └── gram.txt.fsg
│       └── kwslist.txt
│       └── voice_recognizer_pocketsphinx_cmds.yaml
│       └── voice_recognizer_pocketsphinx_kws.yaml
│   └── display_driver.yaml 
│   └── ears_driver.yaml
│   └── neck_driver.yaml
│   └── respeaker_driver.yaml
│   └── robohead_controller.yaml
│   └── sensor_driver.yaml
│   └── speakers_driver.yaml
├── scripts/
│   └── robohead_controller_actions/ # action-скрипты
│       └── std_attention/ # пример содержимого action-скрипта "Внимание"
│            └── action.py # скрипт действия "Внимания"
│            └── attention.mp3 # воспроизводимый звук действия "Внимание"
│            └── attention.png # отображаемая картинка действия "Внимание"
│       └── std_ball_tracker/
│       └── std_ears/
│       └── std_greeting/
│       └── std_attention/
│       └── std_left_ear/
│       └── std_low_bat/
│       └── std_make_photo/
│       └── std_right_ear/
│       └── std_wait/
│   └── loading_splash_1.mp4 # анимация загрузки Робоголовы
│   └── main.py
│   └── robohead_connected.mp3 # озвучивание статуса загрузки Робоголовы
```
---

## Автоматический запуск

* Пакет `robohead_controller` стартует автоматически при загрузке устройства.
* Запуск этого пакета инициирует старты всех зависимых ROS-пакетов (см. `launch/robohead_controller_py.launch` и `launch/dependencies.launch`).

---

## Action-скрипты

При распознавании стандартных голосовых команд вызывается соответствующий action-скрипт:

| Скрипт             | Описание команды                     |
| ------------------ | ------------------------------------ |
| `std_wait`         | Ожидание ключевой фразы              |
| `std_attention`    | «Слушай, Робот!»                     |
| `std_ears`         | «Покажи уши»                         |
| `std_left_ear`     | «Покажи левое ухо»                   |
| `std_right_ear`    | «Покажи правое ухо»                  |
| `std_greeting`     | «Поздоровайся»                       |
| `std_make_photo`   | «Сделай фото»                        |
| `std_ball_tracker` | «Следи за шариком»                   |
| `std_low_bat`      | Низкий заряд — блокирование других скриптов |

> *Расположение:* `scripts/robohead_controller_actions/<script_name>/`

Каждый скрипт содержит код `action.py` и мультимедиа (картинки, аудио, видео). Для замены медиа без изменения кода просто замените файл в папке скрипта, сохранив имя и расширение (для картинок и видео обязательно используйте разрешение 1080x1080 пикселей).

---

## Конфигурация параметров устройства

Основные параметры для подключенных аппаратных компонентов находятся в папке `config/`:

* `display_driver.yaml` — настройка параметров дисплея
* `ears_driver.yaml` — настройка параметров ушных сервоприводов
* `neck_driver.yaml` — настройка параметров шейных сервоприводов
* `respeaker_driver.yaml` — настройка параметров микрофонного массива
* `sensor_driver.yaml` — настройка параметров топика /bat
* `speakers_driver.yaml` — настройка параметров динамиков

Настройка соответствия "голосовая команда - action-скрипт" задается в файле `config/robohead_controller.yaml`.

Подробнее про настройку параметров написано здесь [->](../../30-device-configuration/30-device-setting/30-changing-device-settings.md)