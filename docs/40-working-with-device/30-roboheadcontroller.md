---
id: 30-roboheadcontroller
slug: roboheadcontroller
title: "Описание объектов класса RoboheadController"
sidebar_label: "Описание объектов класса RoboheadController"
sidebar_position: 3
description: "Описание объектов класса RoboheadController"
---

# Описание объектов класса RoboheadController

В разделе опишем класс `RoboheadController` из пакета **robohead_controller**. Именно этот класс используется внутри **action-скриптов** для взаимодействия с различными компонентами Робоголовы. Ниже представлено подробное описание полей и сервисов класса, а также примеры использования.

> **Примечание:** Чтобы action-скрипт смог работать, в начале файла обычно пишут  
> ```python
> from robohead_controller_actions.main import *
> ```  
> Это позволяет импортировать класс `RoboheadController`, передаваемый в функцию  
> `run(robohead_controller: RoboheadController, cmds: str)`.

---

## Введение

Когда вы создаёте **action-скрипт** для Робоголовы, основной точкой входа становится функция:

```python
def run(robohead_controller: RoboheadController, cmds: str):
    # Ваш код здесь
```

* Параметр `robohead_controller` — это экземпляр класса `RoboheadController`, который содержит **все необходимые поля, подписчики и сервисы** для управления Робоголовой.

* Параметр `cmds` — строка с распознанной голосовой командой, по которой запущен скрипт (например, `"улыбнись"` или `"покажи уши"`). Соответствие голосовых команд и скриптов задаётся в файле `robohead_controller.yaml`.

Ниже мы подробно рассмотрим, какие объекты доступны в классе `RoboheadController` и как их использовать.

---

## Сервисы и подписчики

Ниже перечислены **основные доступные поля** класса `RoboheadController`. Для каждого поля указано назначение и пример использования.


### Экран и тачскрин (display_driver)

- **`display_driver_srv_PlayMedia`** (`rospy.ServiceProxy`):  
  Сервис для отображения медиа-файла (изображение или короткое видео) на экране. Ожидает сообщение типа `PlayMediaRequest`, в котором указываются:
  - `path_to_file` — путь к изображению/видео.
  - `is_blocking` — ждать ли окончания показа (1 — да, 0 — нет).
  - `is_cycled` — зацикливать ли показ (0 — один раз, 1 — циклично).

- **`display_driver_pub_PlayMedia`** (`rospy.Publisher`):  
  Публикует сообщения стандартного типа `sensor_msgs/Image` для прямой передачи изображения на экран.

- **`display_driver_sub_touchscreen`** (`rospy.Subscriber`): 
  Подписчик на топик `/robohead_controller/display_driver/touchscreen` с типом данных `geometry_msgs/Pose2D`. 


- **`display_driver_touchscreen_xy`** (`tuple`):  
  Кортеж `(x, y)` с координатами последнего касания на экране.  
  - `display_driver_touchscreen_xy[0]` — X  
  - `display_driver_touchscreen_xy[1]` — Y

> **Пример**: показать картинку и получить координаты касания:
> ```python
> from robohead_controller_actions.main import *
> 
> def run(robohead_controller: RoboheadController, cmds: str):
>     # Получаем путь к текущему скрипту (чтобы корректно ссылаться на медиа-файлы)
>     script_path = os.path.dirname(os.path.abspath(__file__)) + '/'
> 
>     # Показ изображения
>     msg = PlayMediaRequest()
>     msg.path_to_file = script_path + 'image.png'
>     msg.is_blocking = 0
>     msg.is_cycled = 0
>     robohead_controller.display_driver_srv_PlayMedia(msg)
> 
>     # Подождать, чтобы пользователь успел коснуться
>     rospy.sleep(2.0)
>     x, y = robohead_controller.display_driver_touchscreen_xy
>     print(f"Касание: X={x:.1f}, Y={y:.1f}")
> ```
> **Важно:** вывод функции `print()` вы увидете в терминале только, если запустите `robohead_controller` в ручном режиме.

Стандартные действия, в которых можно посмотреть примеры применения:
* [`std_attention`](https://github.com/voltbro/robohead/blob/main/robohead_controller/scripts/robohead_controller_actions/std_attention/action.py)
* [`std_ball_tracker`](https://github.com/voltbro/robohead/blob/main/robohead_controller/scripts/robohead_controller_actions/std_ball_tracker/action.py)
* [`std_ears`](https://github.com/voltbro/robohead/blob/main/robohead_controller/scripts/robohead_controller_actions/std_ears/action.py)
* [`std_greeting`](https://github.com/voltbro/robohead/blob/main/robohead_controller/scripts/robohead_controller_actions/std_greeting/action.py)
* [`std_left_ear`](https://github.com/voltbro/robohead/blob/main/robohead_controller/scripts/robohead_controller_actions/std_left_ear/action.py)
* [`std_make_photo`](https://github.com/voltbro/robohead/blob/main/robohead_controller/scripts/robohead_controller_actions/std_make_photo/action.py)
* [`std_right_earn`](https://github.com/voltbro/robohead/blob/main/robohead_controller/scripts/robohead_controller_actions/std_right_ear/action.py)
* [`std_wait`](https://github.com/voltbro/robohead/blob/main/robohead_controller/scripts/robohead_controller_actions/std_wait/action.py)

---

### Уши и шея (neck_driver, ears_driver)

- **`ears_driver_srv_EarsSetAngle`** (`rospy.ServiceProxy`):  
  Сервис для управления углом поворота ушей. Использует сообщение `EarsSetAngleRequest` с полями:
  - `left_ear_angle` (`float`): угол левого уха (−90° ... +90°).  
  - `right_ear_angle` (`float`): угол правого уха (−90° ... +90°).  

- **`neck_driver_srv_NeckSetAngle`** (`rospy.ServiceProxy`):  
  Сервис для управления шеей. Сообщение `NeckSetAngleRequest` включает:
  - `horizontal_angle` (`int`): горизонтальный угол (−90° ... +90°).  
  - `vertical_angle` (`int`): вертикальный угол (−45° ... +45°).  
  - `duration` (`float`): время в секундах для выполнения поворота.  
  - `is_blocking` (`int`): ждать ли окончания движения (1 — да, 0 — нет).

> **Пример**: движение ушей и поворот головы:
> ```python
> from robohead_controller_actions.main import *
> 
> def run(robohead_controller: RoboheadController, cmds: str):
> 
>     # Поворачиваем уши так, чтобы Робоголова "слушала"
>     msg = EarsSetAngleRequest()
>     msg.left_ear_angle = -45
>     msg.right_ear_angle = 45
>     robohead_controller.ears_driver_srv_EarsSetAngle(msg)
> 
>     # Поворот Робоголовы в сторону говорящего
>     doa = robohead_controller.respeaker_driver_doa_angle
>     angle = min(max(doa, -30), 30)  # ограничим ±30°
>     msg = NeckSetAngleRequest()
>     msg.horizontal_angle = angle
>     msg.vertical_angle = 20
>     msg.duration = 1.0
>     msg.is_blocking = 0
>     robohead_controller.neck_driver_srv_NeckSetAngle(msg)
> ```

Стандартные действия, в которых можно посмотреть примеры применения:
* [`std_attention`](https://github.com/voltbro/robohead/blob/main/robohead_controller/scripts/robohead_controller_actions/std_attention/action.py)
* [`std_ball_tracker`](https://github.com/voltbro/robohead/blob/main/robohead_controller/scripts/robohead_controller_actions/std_ball_tracker/action.py)
* [`std_ears`](https://github.com/voltbro/robohead/blob/main/robohead_controller/scripts/robohead_controller_actions/std_ears/action.py)
* [`std_greeting`](https://github.com/voltbro/robohead/blob/main/robohead_controller/scripts/robohead_controller_actions/std_greeting/action.py)
* [`std_left_ear`](https://github.com/voltbro/robohead/blob/main/robohead_controller/scripts/robohead_controller_actions/std_left_ear/action.py)
* [`std_make_photo`](https://github.com/voltbro/robohead/blob/main/robohead_controller/scripts/robohead_controller_actions/std_make_photo/action.py)
* [`std_right_earn`](https://github.com/voltbro/robohead/blob/main/robohead_controller/scripts/robohead_controller_actions/std_right_ear/action.py)
* [`std_wait`](https://github.com/voltbro/robohead/blob/main/robohead_controller/scripts/robohead_controller_actions/std_wait/action.py)

---

### Динамики и воспроизведение аудио (speakers_driver)

- **`speakers_driver_srv_PlayAudio`** (`rospy.ServiceProxy`):  
  Сервис для воспроизведения аудиофайла. Использует `PlayAudioRequest` с полями:
  - `path_to_file` (`str`): путь к файлу (`.mp3`, `.wav`). Оставьте пустым `""` для принудительной остановки воспроизведения.  
  - `is_blocking` (`int`): ждать ли окончания (1 — да, 0 — нет).  
  - `is_cycled` (`int`): зацикливать ли воспроизведение (1 — да, 0 — нет).  

- **`speakers_driver_srv_GetVolume`** (`rospy.ServiceProxy`):  
  Сервис для получения текущей громкости. Возвращает значение текущей громкости (0...100).

- **`speakers_driver_srv_SetVolume`** (`rospy.ServiceProxy`):  
  Сервис для установки уровня громкости. Использует `SetVolumeRequest`:
  - `volume` (`int`): новое значение (0 ... 100).

> **Пример**: Установка громкости и воспроизведение аудио-файла:
> ```python
> from robohead_controller_actions.main import *
> 
> def run(robohead_controller: RoboheadController, cmds: str):
>     
>     # Получаем путь к текущему скрипту (чтобы корректно ссылаться на медиа-файлы)
>     script_path = os.path.dirname(os.path.abspath(__file__)) + '/'
> 
>     # Установка громкости 50%
>     vol_msg = SetVolumeRequest()
>     vol_msg.volume = 50
>     robohead_controller.speakers_driver_srv_SetVolume(vol_msg)
> 
>     # Воспроизведения аудио-файла
>     msg = PlayAudioRequest()
>     msg.path_to_file = script_path + 'image.png'
>     msg.is_blocking = 1
>     msg.is_cycled = 0
>     robohead_controller.speakers_driver_srv_PlayAudio(msg)
> ```


Стандартные действия, в которых можно посмотреть примеры применения:
* [`std_attention`](https://github.com/voltbro/robohead/blob/main/robohead_controller/scripts/robohead_controller_actions/std_attention/action.py)
* [`std_ball_tracker`](https://github.com/voltbro/robohead/blob/main/robohead_controller/scripts/robohead_controller_actions/std_ball_tracker/action.py)
* [`std_ears`](https://github.com/voltbro/robohead/blob/main/robohead_controller/scripts/robohead_controller_actions/std_ears/action.py)
* [`std_greeting`](https://github.com/voltbro/robohead/blob/main/robohead_controller/scripts/robohead_controller_actions/std_greeting/action.py)
* [`std_left_ear`](https://github.com/voltbro/robohead/blob/main/robohead_controller/scripts/robohead_controller_actions/std_left_ear/action.py)
* [`std_right_earn`](https://github.com/voltbro/robohead/blob/main/robohead_controller/scripts/robohead_controller_actions/std_right_ear/action.py)

---

### Микрофонный массив и светодиодное кольцо (respeakers_driver)

#### Потоки аудио-каналов

- **`respeaker_driver_sub_audio_main`**  
  - **Тип:** `rospy.Subscriber`  
  - **Топик:** основной аудио-канал с микрофона  
  - **Тип сообщения:** `audio_common_msgs/AudioData`

- **`respeaker_driver_sub_audio_channel_{0..5}`**  
  - **Тип:** `rospy.Subscriber`  
  - **Описание:** Подписчики на отдельные аудио-каналы (0, 1, 2, 3, 4, 5).  
  - **Тип сообщения:** `audio_common_msgs/AudioData`

- **`respeaker_driver_msg_audio_main`**  
  - **Тип:** `AudioData`  
  - **Описание:** Последний полученный пакет данных из основного аудио-канала. Поле обновляется автоматически при поступлении сообщений.

- **`respeaker_driver_msg_channel_{0..5}`**  
  - **Тип:** `AudioData`  
  - **Описание:** Последние данные каждого отдельного аудио-канала (0–5).

#### Определение направления (DOA)

1. **`respeaker_driver_sub_doa_angle`**  
   - **Тип:** `rospy.Subscriber`  
   - **Топик:** `/robohead_controller/respeaker_driver/doa_angle`
   - **Тип сообщения:** `std_msgs/Int16` (угол, откуда пришёл звук, в градусах)

2. **`respeaker_driver_doa_angle`**  
   - **Тип:** `int`  
   - **Описание:** Значение последнего угла (DOA — Direction Of Arrival) в градусах. Обновляется автоматически.  

> **Пример**: Вывод угла направления звука
> ```python
> from robohead_controller_actions.main import *
> 
> def run(robohead_controller: RoboheadController, cmds: str):
>     
>    direction = robohead_controller.respeaker_driver_doa_angle
>    rospy.loginfo(f"Звук пришёл с угла: {direction}°")
> ```
> Чтобы увидеть вывод в терминале, запустите `robohead_controller` в ручном режиме.

Стандартное действие, в котором можно посмотреть пример применения:
* [`std_attention`](https://github.com/voltbro/robohead/blob/main/robohead_controller/scripts/robohead_controller_actions/std_attention/action.py)

---

#### Светодиодное кольцо

1. **Параметры по умолчанию**  
   - **`respeaker_driver_default_led_brightness`** (int, 0…31) — яркость  
   - **`respeaker_driver_default_led_mode`** (int) — режим работы светодиодного кольца  
   - **`respeaker_driver_default_led_A_color`** (list из трёх int) — цвет A палитры  
   - **`respeaker_driver_default_led_B_color`** (list из трёх int) — цвет B палитры  

   Значения берутся из конфигурационного файла `robohead_controller/config/respeaker_driver.yaml` при инициализации `RoboheadController`.

2. **`respeaker_driver_pub_SetColorManualLED`**  
   - **Тип:** `rospy.Publisher`  
   - **Тип сообщения:** кастомный тип `respeaker_driver/msg/SetColorManualLED.msg` (массив из 12 стандартных `std_msgs/ColorRGBA`)
   - **Описание:** Позволяет вручную установить цвет и порядок свечения отдельных светодиодов.

3. **`respeaker_driver_srv_SetBrightnessLED`**  
   - **Тип:** `rospy.ServiceProxy`  
   - **Тип сообщения:** кастомный тип `respeaker_driver/stv/SetBrightnessLED.srv` 
   - **Описание:** Сервис для установки яркости всего светодиодного кольца.  
> **Пример**: Установка яркости светодиодного кольца
> ```python
> from robohead_controller_actions.main import *
> 
> def run(robohead_controller: RoboheadController, cmds: str):
>     
>     req = SetBrightnessLEDRequest()
>     req.brightness = 15  # средний уровень
>     robohead_controller.respeaker_driver_srv_SetBrightnessLED(req)
> ```

4. **`respeaker_driver_srv_SetColorAllLED`**  
   - **Тип:** `rospy.ServiceProxy`  
   - **Описание:** Сервис для установки одного цвета для всех светодиодов. Сообщение `SetColorAllLEDRequest` содержит поля `r`, `g`, `b`.  
> **Пример**: Установка синего цвета на всё светодиодное кольцо
> ```python
> from robohead_controller_actions.main import *
> 
> def run(robohead_controller: RoboheadController, cmds: str):
>     
>     req = SetColorAllLEDRequest()
>     req.r = 0; req.g = 0; req.b = 255  # синий цвет для всего кольца
>     robohead_controller.respeaker_driver_srv_SetColorAllLED(req)
>     rospy.sleep(3)
> ````

5. **`respeaker_driver_srv_SetColorPaletteLED`**  
   - **Тип:** `rospy.ServiceProxy`  
   - **Описание:** Сервис для установки цветов палитры для стандартных анимаций светодиодного кольца (например, следящий за направленим звука светодиод)
> **Пример** 
> ```python
> from robohead_controller_actions.main import *
> 
> def run(robohead_controller: RoboheadController, cmds: str):
>     
>     req = SetColorPaletteLEDRequest()
>     req.colorA = [0, 0, 255]  # синий для светодиода, следящего за направлением звука
>     req.colorB = [0, 255, 0]  # зелёный для остальных светодиодов
>     robohead_controller.respeaker_driver_srv_SetColorPaletteLED(req)
>     rospy.sleep(5)
> ```

6. **`respeaker_driver_srv_SetModeLED`**  
   - **Тип:** `rospy.ServiceProxy`  
   - **Описание:** Сервис для переключения режима работы светодиодного кольца. 
> **Пример**     
> ```python
> from robohead_controller_actions.main import *
> 
> def run(robohead_controller: RoboheadController, cmds: str):
>     
>     req = SetModeLEDRequest()
>     req.mode = 3 
>     robohead_controller.respeaker_driver_srv_SetModeLED(req)
>     rospy.sleep(5)
> ```

Стандартные действия, в которых можно посмотреть примеры применения:
* [`std_attention`](https://github.com/voltbro/robohead/blob/main/robohead_controller/scripts/robohead_controller_actions/std_attention/action.py)
* [`std_greeting`](https://github.com/voltbro/robohead/blob/main/robohead_controller/scripts/robohead_controller_actions/std_greeting/action.py)
* [`std_wait`](https://github.com/voltbro/robohead/blob/main/robohead_controller/scripts/robohead_controller_actions/std_wait/action.py)

---

### Распознавание речи (voice_recognizer_pocketsphinx)

1. **`voice_recognizer_pocketsphinx_sub_kws`**  
   - **Тип:** `rospy.Subscriber`  
   - **Топик:** получение распознанных ключевых слов (`/robohead_controller/voice_recognizer_pocketsphinx/kws_recognizer/keywords`) тип данных `std_msgs/String`  
   - **Описание:** Позволяет обрабатывать короткие ключевые слова (например, «слушайробот»).

2. **`voice_recognizer_pocketsphinx_sub_cmds`**  
   - **Тип:** `rospy.Subscriber`  
   - **Топик:** получение распознанных команд (`/robohead_controller/voice_recognizer_pocketsphinx/cmds_recognizer/commands`) тип данных `std_msgs/String`
   - **Описание:** Используется для получения полной голосовой команды (например, «покажи уши»).

3. **`voice_recognizer_pocketsphinx_kws_srv_IsWork`**  
   - **Тип:** `rospy.ServiceProxy`  
   - **Описание:** Сервис для включения/отключения распознавания ключевых слов (тип сообщения `voice_recognizer_pocketsphinx/IsWork`).  
> **Пример**     
> ```python
> from robohead_controller_actions.main import *
> 
> def run(robohead_controller: RoboheadController, cmds: str):
>     
>     req = IsWorkRequest()
>     req.SetStatus = False  # временно отключаем распознавание ключевых слов
>     robohead_controller.voice_recognizer_pocketsphinx_kws_srv_IsWork(req)
>     rospy.sleep(3)
> ```

4. **`voice_recognizer_pocketsphinx_cmds_srv_IsWork`**  
   - **Тип:** `rospy.ServiceProxy`  
   - **Описание:** Сервис для включения/отключения распознавания голосовых команд (полных фраз).
> **Пример**     
> ```python
> from robohead_controller_actions.main import *
> 
> def run(robohead_controller: RoboheadController, cmds: str):
>     
>     req = IsWorkRequest()
>     req.SetStatus = True   # включаем распознавание фраз
>     robohead_controller.voice_recognizer_pocketsphinx_cmds_srv_IsWork(req)
>     rospy.sleep(3)
> ```  

---

### USB-камера (usb_cam)

- **`usb_cam_sub_image_raw`** (`rospy.Subscriber`):  
  Подписчик на топик с изображениями с USB-камеры. Тип сообщения — `sensor_msgs/Image`.

- **`usb_cam_image_raw`** (`Image`):  
  Текущее изображение с камеры. Автоматически обновляется при получении нового кадра.

> **Пример**: вывести размеры изображения в пикселях:
> ```python
> from robohead_controller_actions.main import *
> 
> def run(robohead_controller: RoboheadController, cmds: str):
>     
>     image_msg = robohead_controller.usb_cam_image_raw
>     width = image_msg.width
>     height = image_msg.height
>     rospy.loginfo(f"Размер изображения: {width}x{height}")
> ``` 
> Чтобы увидеть вывод размера изображения в терминале запустите `robohead_controller`  в ручном режиме

Стандартные действия, в которых можно посмотреть примеры применения:
* [`std_ball_tracker`](https://github.com/voltbro/robohead/blob/main/robohead_controller/scripts/robohead_controller_actions/std_ball_tracker/action.py)
* [`std_make_photo`](https://github.com/voltbro/robohead/blob/main/robohead_controller/scripts/robohead_controller_actions/std_make_photo/action.py)

---

### Датчик тока и напряжения аккумулятора (sensor_driver) 

- **`sensor_driver_bat_current`** (`float`):  
  Текущий ток, потребляемый от батареи (в амперах). Значение переменной обновляется автоматически.

- **`sensor_driver_bat_voltage`** (`float`):  
  Текущее напряжение батареи (в вольтах). Значение переменной обновляется автоматически.

- **`sensor_driver_sub_battery`** (`rospy.Subscriber`):  
  Подписчик на топик `/robohead_controller/sensor_driver/bat` с типом данных `sensor_msgs/BatteryState`.

> **Пример**: проверка напряжения и вывод предупреждения, если батарея разряжена:
> ```python
> from robohead_controller_actions.main import *
>
> def run(robohead_controller: RoboheadController, cmds: str):
>
>     voltage = robohead_controller.sensor_driver_bat_voltage
>     current = robohead_controller.sensor_driver_bat_current
>     if voltage < 3.8:
>         print(f"Низкий уровень заряда: {voltage:.2f} В, ток: {current:.2f} А")
> ```
> Чтобы увидеть вывод функции `print(...)` в терминале запустите `robohead_controller`  в ручном режиме

---

### Конфигурационные параметры (из robohead_controller.yaml)

Эти параметры лучше изменять в конфигурационном файле непосредственно. Из кода программы их желательно только читать.

1. **`robohead_controller_low_voltage_threshold`**  
   - **Тип:** `float`  
   - **Источник:** параметр `low_voltage_threshold` из `robohead_controller.yaml`  
   - **Описание:** Граница низкого напряжения, ниже которой Робоголове запрещается запускать действия.

2. **`robohead_controller_low_voltage_hysteresis`**  
   - **Тип:** `float`  
   - **Источник:** параметр `low_voltage_hysteresis` из `robohead_controller.yaml`  
   - **Описание:** Гистерезис для порога низкого напряжения (чтобы избежать «дрожания» состояния при значениях рядом с порогом).

3. **`robohead_controller_actions_match`**  
   - **Тип:** `dict`  
   - **Источник:** параметр `robohead_controller_actions_match` из `robohead_controller.yaml`  
   - **Описание:** Словарь, где ключ — строка голосовой команды, а значение — путь к модулю скрипта (например, `"улыбнись": "robohead_controller_actions.smile.action"`).

4. **`robohead_controller_is_allow_work`**  
   - **Тип:** `bool`  
   - **Описание:** Флаг, разрешающий или запрещающий выполнение action-скриптов. Устанавливается автоматически на основе текущего напряжения (`sensor_driver_bat_voltage`) и пороговых значений `low_voltage_threshold`/`low_voltage_hysteresis`.

> **Пример использования:**  
> ```python
> from robohead_controller_actions.main import *
> 
> def run(robohead_controller: RoboheadController, cmds: str):
>     
>     if not robohead_controller.robohead_controller_is_allow_work:
>         rospy.logwarn("Выполнение скриптов запрещено: низкое напряжение.")
>         return
> ``` 

---

## Примеры использования

Ниже — несколько **полных примеров** для демонстрации возможностей класса.

### Пример 1. Проверка батареи и отображение предупреждения

```python
def run(robohead_controller: RoboheadController, cmds: str):
    voltage = robohead_controller.sensor_driver_bat_voltage
    if voltage < robohead_controller.robohead_controller_low_voltage_threshold:
        # Включаем предупреждение звуком
        audio_msg = PlayAudioRequest()
        audio_msg.path_to_file = "/home/pi/robohead_ws/src/robohead/robohead_controller/scripts/robohead_controller_actions/std_low_bat/low_bat.mp3"
        audio_msg.is_blocking = 0
        audio_msg.is_cycled = 0
        robohead_controller.speakers_driver_srv_PlayAudio(audio_msg)

        # Показываем картинку «низкий заряд»
        img_msg = PlayMediaRequest()
        img_msg.path_to_file = "/home/pi/robohead_ws/src/robohead/robohead_controller/scripts/robohead_controller_actions/std_low_bat/low_bat.png"
        img_msg.is_blocking = 0
        img_msg.is_cycled = 0
        robohead_controller.display_driver_srv_PlayMedia(img_msg)
```

### Пример 2. Ожидание касания экрана для завершения действия

```python
def run(robohead_controller: RoboheadController, cmds: str):
  rospy.loginfo("Показываем сообщение на экране. Ожидаем касания...")
  msg = PlayMediaRequest()
  msg.path_to_file = "/home/pi/robohead_ws/src/robohead/robohead_controller/scripts/robohead_controller_actions/std_greeting/greeting.png"
  msg.is_blocking = 0
  msg.is_cycled = 0
  robohead_controller.display_driver_srv_PlayMedia(msg)

  start_x, start_y = robohead_controller.display_driver_touchscreen_xy
  x, y = robohead_controller.display_driver_touchscreen_xy
  
  # Ждём касания
  while (x==start_x) and (y==start_y):
      x, y = robohead_controller.display_driver_touchscreen_xy
      rospy.sleep(0.1)

  rospy.loginfo(f"Экрана коснулись: ({x:.1f}, {y:.1f}). Завершаем.")
```


---

## Рекомендации и советы

- **Обработка ошибок и исключений:** Проверяйте доступность сервисов и паблишеров перед их вызовом. .
- **Асинхронные действия:** Если не хотите блокировать главный поток ROS, устанавливайте `is_blocking = 0` и разбивайте длительные операции на более мелкие.
- **Логирование:** Используйте `rospy.loginfo()`, `rospy.logwarn()`, `rospy.logerr()` для отслеживания состояния Робоголовы во время выполнения action-скриптов.
- **Структурирование кода:** Группируйте вызовы, относящиеся к одному компоненту (например, всё, что связано с LED, — в один блок; всё, что связано с аудио — в другой).
- **Тестирование:** Периодически проверяйте работу каждого сервиса в отдельности, запуская `robohead_controller` в режиме отладки:
  ```bash
  # Останавливаем Ubuntu-сервис
  sudo systemctl stop robohead.service

  # Запускаем robohead_controller в ручном режиме
  roslaunch robohead_controller robohead_controller_py.launch
  ```

---

## Заключение

Класс `RoboheadController` предоставляет **богатый набор инструментов** для создания интерактивных действий Робоголовы. С его помощью можно:

- Отслеживать состояние батареи и принимать решения при низком напряжении.  
- Отображать изображения и обрабатывать касания на экране.  
- Управлять механикой ушей и шеи Робоголовы.  
- Воспроизводить звуковые сигналы и настраивать громкость.  
- Собирать данные с микрофона (ReSpeaker), в том числе направление звука.  
- Обрабатывать видео с USB-камеры.  
- Управлять режимом распознавания голоса (включать/выключать).

Используя примеры выше, вы сможете быстро начать разработку собственных **action-скриптов**, создающих уникальную логику поведения вашей Робоголовы.