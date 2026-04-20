---
id: 40-сhanging-standart-media-files
title: Смена медиафайлов в стандартных командах
---

# Смена медиафайлов в стандартных командах

В Робоголове Bbrain стандартные голосовые команды сопровождаются медиафайлами (изображения, аудио), расположенными в папке `robohead_controller/robohead_controller/actions/<имя_скрипта>/`:

```
robohead_controller
└── robohead_controller
    └── actions
        ├── std_attention
        │   ├── action.py
        │   ├── attention.gif
        │   ├── attention.mp3
        │   ├── attention.mp4
        │   └── attention.png
        ├── std_ball_tracker
        │   ├── action.py
        │   ├── calibrate_voice.mp3
        │   └── finish_voice.mp3
        ├── std_ears
        │   ├── action.py
        │   ├── ears.mp3
        │   └── ears.png
        ├── std_echo
        │   ├── action.py
        │   ├── microphone.gif
        │   ├── microphone.mp4
        │   ├── speaker.gif
        │   └── speaker.mp4
        ├── std_greeting
        │   ├── action.py
        │   ├── greeting.gif
        │   ├── greeting.mp3
        │   ├── greeting.mp4
        │   └── greeting.png
        ├── std_left_ear
        │   ├── action.py
        │   ├── left_ear.mp3
        │   └── left_ear.png
        ├── std_llm
        │   ├── action.py
        │   ├── loading.gif
        │   ├── microphone.mp4
        │   └── speaker.mp4
        ├── std_low_bat
        │   ├── action.py
        │   ├── low_bat.gif
        │   ├── low_bat.mp3
        │   └── low_bat.mp4
        ├── std_make_photo
        │   ├── action.py
        │   ├── make_photo.mp3
        │   └── make_photo.png
        ├── std_right_ear
        │   ├── action.py
        │   ├── right_ear.mp3
        │   └── right_ear.png
        ├── std_show_voltage
        │   └── action.py
        ├── std_startup
        │   ├── action.py
        │   ├── hello.gif
        │   └── hello.mp4
        ├── std_volume_down
        │   ├── action.py
        │   └── set_vol.mp3
        ├── std_volume_up
        │   ├── action.py
        │   ├── max_vol.mp3
        │   └── set_vol.mp3
        └── std_wait
            ├── action.py
            ├── wait.mp4
            └── wait.png

```

---

## Структура папки с командами

Например, для команды «Поздоровайся» папка выглядит так:
```
std_greeting
├── action.py
├── greeting.gif
├── greeting.mp3
├── greeting.mp4
└── greeting.png
```

* `action.py` — код выполнения действия
* `<имя>.mp3/.wav/.mp4` — звуковое сопровождение
* `<имя>.png/.jpg/.mp4/.gif` — изображение/видео для вывода на экран

---

## 2. Порядок замены медиафайлов

1. **Подготовьте новые файлы**

   * *Аудио:* `.mp3` или `.wav`
   * *Изображения:* `.png` или `.jpg`, размер строго **1080×1080** пикселей (или видео `.mp4`)

2. **Переименуйте файлы**
   Замените старые файлы, сохранив имена. Например, `greeting.mp3` → `greeting.mp3`.

3. **Замените в папке**
   Скопируйте новые файлы в `robohead_controller/robohead_controller/actions/<имя_скрипта>/`, перезаписав старые.

4. **Примененине изменений**
   Изменения в action-действиях происходят сразу после сохранения файлов. Перезапускать пакет `robohead_controller` (и все сопутствующие зависимости) не нужно
> Перезапуск `robohead_controller` необходим только в случае изменения файловой структуры: добавление новых картинок, добавление новых действий (папок)
---

## 3. Проверка изменений

- Произнесите команду, например, «Поздоровайся».  
- Убедитесь, что на экране выводится новое изображение/видео и воспроизводится новое аудио.

---

## 4. Советы

- Храните резервные копии оригинальных файлов.  
- Проверяйте формат и разрешение медиа до замены.
- Если вы хотите сгенерировать файлы с озвучиванием текста, то рекомендуем воспользоваться сервисом [VoiceBot](https://voicebot.su/)

    ### Параметры `VoiceBot`

    - **Голос**: Антон
    - **Скорость**: 0.9
    - **Высота**: 0.0
    - **Громкость**: 0 dB
    - **Эмоции**: Радостный
