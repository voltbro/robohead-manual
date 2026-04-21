---
id: 10-changing-standart-action
slug: changing-standart-action
title: "Изменение действий в стандартных командах"
sidebar_label: "Изменение действий в стандартных командах"
sidebar_position: 1
description: "Изменение действий в стандартных командах"
---

# Внесение изменений в стандартные действия

Стандартные действия — это готовые сценарии, которые позволяют Робоголове реагировать на голосовые команды, показывать мультимедиа и выполнять простые движения. Но всегда возникает желание добавить чего-то своего: новую картинку, другую мелодию, изменить угол наклона головы или даже добавить дополнительный эффект. 

Рассмотрим настройку стандартных действий на примере стандартного действия `std_ears`, которое вызывается по команде "Слушай, Робот! Покажи уши".

---

:::warning
Любые изменения в **коде (`action.py`)** стандартных действий применяются "на лету", после сохранения изменений файлов. Если началось непредсказуемое поведение - перезапустите сервис `robohead.service` или запустите пакет `robohead_controller` в ручном режиме (см. далее).
:::


## 1. Расположение файлов стандартных действий

Стандартные действия находятся в папке `~/robohead_ws/src/robohead2/robohead_controller/robohead_controller/actions` пакета [`robohead_controller`](../10-introduction/40-software/41-robohead-controller.md). Для `std_ears` перейдите в папку:
```bash
cd ~/robohead_ws/src/robohead2/robohead_controller/robohead_controller/actions/std_ears
```

В ней вы увидите три основных файла:
- **`ears.png`** — картинка с ушами, которая выводится на экран;
- **`ears.mp3`** — аудиофайл, который воспроизводится через динамики;
- **`action.py`** — скрипт, описывающий всю логику действия: что выводить на экран/динамики, чем и куда двигать, в какой последовательности.

Пример структуры папки:
```
std_ears/
├── ears.png
├── ears.mp3
└── action.py
```

---

## 2. Разбор скрипта `action.py`

Откройте файл `action.py`. Основной блок кода выглядит примерно так:

```python
# std_ears
# действие, выполняющееся при команде "Покажи уши"

# Импорты нужны для автоподстановок кода при работе через VSCode
from __future__ import annotations
from typing import TYPE_CHECKING
import os

if TYPE_CHECKING:
    from robohead_controller.controller import RoboheadController
    import threading


def run(
    controller: RoboheadController, action_name: str, cancel_event: threading.Event
):
    """
    Args:
        controller: Ссылка на контроллер
        action_name: Команда, по которой было вызвано действие
        cancel_event: threading.Event для проверки отмены
    """
    action_dir = os.path.dirname(os.path.abspath(__file__)) # Путь к папке со скриптом, обычно это:
    # /home/pi/robohead_ws/build/robohead_controller/robohead_controller/actions/std_ears

    logger = controller.get_logger()        # logger - объект логирования, через него можно печатать в консоль
    logger.info(f"[{action_name}] start")   # выводим в терминал "[std_ears] start"

    # Выводим картинку ears.png без зацикливания воспроизведения (это же картинка) и блокирования вызова
    controller.media_driver.play_display(
        cancel_event=cancel_event,
        video_path=os.path.join(action_dir, "ears.png"),
        loop=False,
        block=False,
    )

    # Проигрываем звук ears.mp3 без зацикливания воспроизведения и блокирования вызова
    controller.media_driver.play_audio(
        cancel_event=cancel_event,
        audio_path=os.path.join(action_dir, "ears.mp3"),
        loop=False,
        block=False,
    )

    for k in range(5): # Цикл 5 раз
        # Поворачиваем голову
        controller.neck_driver.set_angle(
            cancel_event=cancel_event,
            horizontal=15 * (-1) ** k,  # значения будут: 15, -15, 15, -15, 15
            vertical=15,    # Вертикальный подьем головы 15 градусов
            duration=0.5,   # Длительность достижения заданной позиции 0.5 секунд
            block=False,    # Вызов без блокирования
        )
        # Поворачиваем уши
        controller.ears_driver.set_angle(
            cancel_event=cancel_event,
            left=90 * (-1) ** k,    # Значения поворота левого уха: 90, -90, 90, -90, 90
            right=-90 * (-1) ** k,  # Значения поворота правого уха: -90, 90, -90, 90, -90
            duration=0.5,   # Длительность достижения заданной позиции 0.5 секунд
            block=True,     # Блокирующий вызов: программа здесь "зависнет" на 0.5 секунд (duration)
        )

    logger.info(f"[{action_name}] finish")  # выводим в терминал "[std_ears] finish"
```
Это был полный код. Условно алгоритм выглядит так:
```python
# std_ears
# действие, выполняющееся при команде "Покажи уши"

def run(controller: RoboheadController, action_name: str, cancel_event: threading.Event):
    # Выводим картинку ears.png без зацикливания воспроизведения (это же картинка) и блокирования вызова
    controller.media_driver.play_display(...)

    # Проигрываем звук ears.mp3 без зацикливания воспроизведения и блокирования вызова
    controller.media_driver.play_audio(...)

    for k in range(5): # Цикл 5 раз
        # Поворачиваем голову
        controller.neck_driver.set_angle(...)

        # Поворачиваем уши
        controller.ears_driver.set_angle(...)

```

---

## 3. Примеры пользовательских изменений

Ниже приведены несколько примеров, как разнообразить действие `std_ears` без глубокого изменения логики.

### 3.1 Замена медиа-ресурсов

- **Вывод видео вместо статичного изображения**  
  1. В папке `std_ears` уже лежит файл `video.mp4`. (При желании вы можете заменить его на свой) 
  2. В файле `action.py` измените имя файла, который нужно вывести на дисплей :

  Было:
  ```
   controller.media_driver.play_display(
       cancel_event=cancel_event,
       video_path=os.path.join(action_dir, "ears.png"),
       loop=False,
       block=False,
   )
  ```
  Стало:
  ```
   controller.media_driver.play_display(
       cancel_event=cancel_event,
       video_path=os.path.join(action_dir, "video.mp4"),
       loop=False,
       block=False,
   )
  ```
  После этого скажите голосом: *"Слушай, Робот! Покажи уши"*, и робот покажет видео на экране.

- **Изменение звукового сопровождения**  

Бывает хочется, чтобы Робоголова не просто проигрывала короткий звук «Ухи мои ухи», а, например, сначала здоровалась. Для этого вы можете либо записать аудио-файл (.mp3/.wav) и воспроизвести его, либо запусить синтез речи и Робоголова озучит ваш текст сама.

**Вариант 1 (воспроизведение готового аудио-файла)**

1. В папке `std_ears` уже лежит файл `audio.mp3`. (При желании вы можете заменить его на свой) 
2. Вставьте в код `action.py` перед вызовом `controller.media_driver.play_audio(...)` такой же вызов, но с файлом `audio.mp3` и укажите параметр `block=true`. Получится:
   ```python
    controller.media_driver.play_audio(
        cancel_event=cancel_event,
        audio_path=os.path.join(action_dir, "audio.mp3"),
        loop=False,
        block=True,
    )

    # Проигрываем звук ears.mp3 без зацикливания воспроизведения и блокирования вызова
    controller.media_driver.play_audio(
        cancel_event=cancel_event,
        audio_path=os.path.join(action_dir, "ears.mp3"),
        loop=False,
        block=False,
    )
   ```

3. Обратите внимание, что **`block = True`** означает ожидание завершения воспроизведения аудио перед продолжением выполнения скрипта.

**Вариант 2 (синтез речи)**
1. Вставьте в код `action.py` перед вызовом `controller.media_driver.play_audio(...)` вызов `silero_tts.say(...)` с текстом для озвучки. Должно получиться:
    ```python
    controller.silero_tts.say(cancel_event=cancel_event, text="Добрый день. Сейчас я покажу вам свои уши!")

    # Проигрываем звук ears.mp3 без зацикливания воспроизведения и блокирования вызова
    controller.media_driver.play_audio(
        cancel_event=cancel_event,
        audio_path=os.path.join(action_dir, "ears.mp3"),
        loop=False,
        block=False,
    )
    ```

### 3.2 Настройка положения шеи

- **Изменение угла обзора**  
  Для более выразительной реакции можно поднять голову выше или опустить:
  ```diff
  controller.neck_driver.set_angle(
      cancel_event=cancel_event,
      horizontal=0 # Фиксированное положение по горизонтали 0 градусов (голова смотрит вперед)
      vertical=-15,    # Вертикальное опускание головы 15 градусов
      duration=0.5,   # Длительность достижения заданной позиции 0.5 секунд
      block=False,    # Вызов без блокирования
  )
  ```
- **Изменение скорости движения**  
  Параметр `duration` отвечает за время выполнения движения. Меньшее значение — более резкое движение, большее - более плавное движение:
  ```diff
  controller.neck_driver.set_angle(
      cancel_event=cancel_event,
      horizontal=0 # Фиксированное положение по горизонтали 0 градусов (голова смотрит вперед)
      vertical=-15,    # Вертикальное опускание головы 15 градусов
      duration=1.0,   # Длительность достижения заданной позиции за 1.0 секунду (вместо 0.5)
      block=False,    # Вызов без блокирования
  )
  ```
- **Добавление паузы перед движением**  
  Чтобы голосовая команда сначала воспроизводила звук и изображение, а через 2 секунды двигала шею:
  ```python
  controller.sleep(cancel_event=cancel_event, duration=2.0) # Пауза 2 секунды
  # controller.neck_driver.set_angle(...)
  # ...
  ```

  ### 3.3 Настройка «анимации» ушей

По умолчанию уши подергиваются 5 раз с интервалом 0.5 секунды. Вот несколько примеров изменений:

- **Увеличить количество повторений**  
  Измените `range(5)` на большее значение, например:
  ```diff
  - for k in range(5):
  + for k in range(10):  # Подергивание 10 раз
  ```

- **Увеличить интервал**  
  Чтобы подергивания были медленнее (интервал 1 секунда):
  ```python
  controller.ears_driver.set_angle(
      cancel_event=cancel_event,
      left=90 * (-1) ** k,    
      right=-90 * (-1) ** k,  
      duration=1.0,   # Длительность достижения заданной позиции 1.0 секунда
      block=True,     
  )
  ```

- **Сложная анимация**  
  Можно задать последовательность углов в списке:
  ```python
  # Поворачиваем голову
  controller.neck_driver.set_angle(
      cancel_event=cancel_event,
      horizontal=0,
      vertical=15,    
      duration=0.5,  
      block=False,    
  )    
  angles = [30, -30, 45, -45, 90, -90]
  for angle in angles:
      # Поворачиваем уши
      controller.ears_driver.set_angle(
          cancel_event=cancel_event,
          left=angle,   
          right=angle,  
          duration=0.5,   # Длительность достижения заданной позиции 0.5 секунд
          block=True,     # Блокирующий вызов: программа здесь "зависнет" на 0.5 секунд (duration)
      )
  ```

## 4. Запуск пакета `robohead_controller` в ручном режиме

Для тестирования бывает удобно запустить пакет `robohead_controller` в ручном режиме, чтобы видеть появляющиеся ошибки и информацию для отладки.

### Шаг 1. Остановка Ubuntu-сервиса

По-умолчанию, пакет `robohead_controller` и все зависимости запускаются автоматически при загрузке Робоголовы при помощи Ubuntu-сервиса. Для отладки и запуска пакета в ручном режиме его необходимо остановить

```bash
sudo systemctl stop robohead.service
```

---

### Шаг 2. Запуск пакета вручную

Запустите пакет отдельно через launch-файл:
```bash
ros2 launch robohead_controller robohead_controller.launch.py
```
На экране терминала должен появиться примерно следующий вывод:

![Запуск без ошибок](attachments/robohead_controller-launch.png)

Дождитесь полного запуска - Робоголова выведет изображение и поздоровается.

## 5. Отладка команд
В этом шаге описано как запускать стандартные действия, не произнося каждый раз голосом нужную команду.

Отлаживать команды можно как при автоматически запущенном `robohead_controller` (через Ubuntu-сервис), так и когда он запущен в ручном режиме (этот вариант предпочтительнее для отладки, так как видны все возникающие ошибки)

Откройте новое окно терминала, подключитесь к Робоголове.
Далее нужно опубликовать Робоголове ключевую фразу:
```bash
ros2 topic pub /robohead/speech_recognizer/kws/wake_phrases std_msgs/msg/String "data: 'слушай робот'" --once
```

И, пока не прошел таймаут, успеть опубликовать команду, которую нужно запустить:
```bash
ros2 topic pub /robohead/speech_recognizer/asr/commands std_msgs/msg/String "data: 'покажи уши'" --once
```

После публикации команды Робоголова должна выполнить стандартное действие "**Покажи уши**".

Аналагично можно запускать и другие стандартные действия.
> В `.../kws/wake_phrases` можно публиковать что угодно - фиксируется факт нахождения ключевой фразы
> В `.../asr/command` нужно публиковать команды, которые описаны в `actions_match` в `robohead_controller/config/robohead_controller.yaml`.