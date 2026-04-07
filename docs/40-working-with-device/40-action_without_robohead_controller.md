---
id: 40-action_without_robohead_controller
slug: action_without_robohead_controller
title: "Создание своих сценариев взаимодействия без использования robohead_controller"
sidebar_label: "Создание своих сценариев взаимодействия без использования robohead_controller"
sidebar_position: 4
description: "Создание своих сценариев взаимодействия без использования robohead_controller"
---


# Создание собственного сценария без использования `robohead_controller`

В этом разделе мы создадим изменённую версию команды **«Улыбнись»**, реализованную без использования оболочки `robohead_controller`. Такой подход позволяет более гибко управлять устройством, вызывая сервисы напрямую из своего скрипта.

---

## Описание задачи

Наша цель — написать скрипт, который выполнит следующие действия:

1. Выведет изображение **`smile.png`** на дисплей Робоголовы.
2. Проиграет аудио-файл **`smile.mp3`** через динамики.
3. На протяжении 3 секунд покачает ушами вперёд-назад с частотой 2 Гц.
4. С помощью шейного сустава поднимет и повернёт голову в сторону.
5. После завершения всех действий вернёт голову и уши в исходное положение.

Такой сценарий позволит познакомиться с низкоуровневым взаимодействием с драйверами Робоголовы, минуя обёртку `robohead_controller`.

---

## Шаг 1. Подготовка

1. Создайте в домашней папке (`home/pi`) папку `scripts` перейдите в неё

2. Скопируйте в папку два медиа-файла:
   - [`smile.png`](attachments/smile.png) — изображение для отображения.
   - [`smile.mp3`](attachments/smile.mp3) — аудиофайл для воспроизведения.

3. В этой же папке создайте файл скрипта `main.py`:

---

## Шаг 2. Структура и содержание скрипта `main.py`

В файле `main.py` мы будем обращаться напрямую к сервисам пакетов Робоголовы: дисплею, динамикам, ушам и шее. Ниже приводём подробный разбор кода.

```python
import os
import rospy

# Импортируем типы запросов и ответов для сервисов дисплея
from display_driver.srv import PlayMedia, PlayMediaRequest

# Импортируем типы запросов и ответов для сервиса управления ушами
from ears_driver.srv import EarsSetAngle, EarsSetAngleRequest

# Импортируем типы запросов и ответов для сервиса управления шеей
from neck_driver.srv import NeckSetAngle, NeckSetAngleRequest

# Импортируем типы запросов и ответов для управления динамиками
from speakers_driver.srv import PlayAudio, PlayAudioRequest, SetVolume, SetVolumeRequest

# Инициализируем ROS-ноду с названием "smile_node"
rospy.init_node("smile_node")
script_path = os.path.dirname(os.path.abspath(__file__)) + '/'
print('Инициализация узла завершена')
```

1. **Инициализация ROS-ноды**  
   Мы вызываем `rospy.init_node("smile_node")`, чтобы зарегистрировать новый узел в ROS. Переменная `script_path` содержит полный путь к папке со скриптом, чтобы затем корректно формировать пути к медиа-файлам.

```python
# Ожидаем, пока все нужные сервисы будут доступны
rospy.wait_for_service('/robohead_controller/display_driver/PlayMedia')
rospy.wait_for_service('/robohead_controller/neck_driver/NeckSetAngle')
rospy.wait_for_service('/robohead_controller/ears_driver/EarsSetAngle')
rospy.wait_for_service('/robohead_controller/speakers_driver/PlayAudio')
rospy.wait_for_service('/robohead_controller/speakers_driver/SetVolume')
print('Все сервисы доступны')
```

2. **Ожидание сервисов**  
   Перед тем как отправлять запросы, необходимо убедиться, что сервисы на стороне пакетов устройств уже запущены и готовы принять команды. Мы дожидаемся следующих сервисов:
   - `/robohead_controller/display_driver/PlayMedia` — воспроизведение изображений.
   - `/robohead_controller/neck_driver/NeckSetAngle` — управление углами шеи.
   - `/robohead_controller/ears_driver/EarsSetAngle` — управление углами ушей.
   - `/robohead_controller/speakers_driver/PlayAudio` — воспроизведение аудио.
   - `/robohead_controller/speakers_driver/SetVolume` — установка громкости.

```python
# Создаём объекты-посредники (ServiceProxy) для каждого сервиса
service_display = rospy.ServiceProxy(
    '/robohead_controller/display_driver/PlayMedia', PlayMedia
)
service_neck = rospy.ServiceProxy(
    '/robohead_controller/neck_driver/NeckSetAngle', NeckSetAngle
)
service_ears = rospy.ServiceProxy(
    '/robohead_controller/ears_driver/EarsSetAngle', EarsSetAngle
)
service_play_audio = rospy.ServiceProxy(
    '/robohead_controller/speakers_driver/PlayAudio', PlayAudio
)
service_set_volume = rospy.ServiceProxy(
    '/robohead_controller/speakers_driver/SetVolume', SetVolume
)
print('ServiceProxy для всех сервисов создан')
print('Запуск сценария "Улыбнись"')
```

3. **Создание объектов ServiceProxy**  
   При помощи `rospy.ServiceProxy` мы создаём вызовы к сервисам, используя их имена и типы сообщений.

---

## Шаг 3. Пошаговое выполнение сценария

### 3.1. Показ изображения на дисплее

```python
# Заполняем запрос для воспроизведения изображения на экране
msg = PlayMediaRequest()
msg.path_to_file = script_path + 'smile.png'
msg.is_blocking = 0    
msg.is_cycled = 0     
service_display(msg)
print('Изображение отображено')
```

### 3.2. Поворот шеи и наклон головы

```python
# Подготовим запрос для изменения угла шеи
neck_msg = NeckSetAngleRequest()
neck_msg.horizontal_angle = 30  # поворот вправо на 30°
neck_msg.vertical_angle = 15    # поднятие вверх на 15°
neck_msg.is_blocking = 0       
neck_msg.duration = 1           
service_neck(neck_msg)
print('Шея повернута: горизонтальный угол 30°, вертикальный угол 15°')
```

- **`horizontal_angle`** — угол поворота шеи в горизонтальной плоскости (градусы).
- **`vertical_angle`** — угол наклона шеи в вертикальной плоскости (градусы).
- **`duration`** — время, за которое будет выполнено движение (в секундах).

### 3.3. Установка громкости и воспроизведение звука

```python
# Устанавливаем громкость на 60% из диапазона 0..100
volume_msg = SetVolumeRequest()
volume_msg.volume = 60
service_set_volume(volume_msg)
print('Установлена громкость 60%')

# Воспроизводим аудио-файл
audio_msg = PlayAudioRequest()
audio_msg.path_to_file = script_path + 'smile.mp3'
audio_msg.is_cycled = 0   
audio_msg.is_blocking = 0 
service_play_audio(audio_msg)
print('Аудио "smile.mp3" запущено')
```

- **`volume_msg.volume`** — значение громкости от 0 до 100.
- **`audio_msg.path_to_file`** — путь к mp3-файлу.

### 3.4. Анимация ушей

Для создания эффекта покачивания ушей мы будем попеременно устанавливать углы на 45° вперёд и назад с шагом 0.5 секунды в течение 3 секунд.

```python
# Начальное время для отсчёта трёх секунд
start_time = rospy.get_time()

# Начальное направление движения ушей: 1 — вперёд, -1 — назад
direction = 1
ears_msg = EarsSetAngleRequest()

# Пока не прошло 3 секунды, дергаем ушами
while (rospy.get_time() - start_time) < 3:
    ears_msg.left_ear_angle = direction * 45   # угол для левого уха
    ears_msg.right_ear_angle = -direction * 45 # угол для правого уха
    service_ears(ears_msg)
    direction *= -1
    rospy.sleep(0.5)

print('Анимация ушей (3 секунды) завершена')
```

- **`ears_msg.left_ear_angle`** и **`ears_msg.right_ear_angle`** — углы поворота левого и правого уха (в градусах).
- Мы меняем направление (`direction *= -1`) каждые 0.5 секунды.

### 3.5. Завершение сценария: возврат в исходное положение

После того, как изображение и звук уже проиграны, а анимация с ушами завершена, необходимо убрать картинку с дисплея и вернуть голову и уши в нулевое положение.

```python
# Отключаем отображение изображения (отправляем пустой вызов)
service_display()

# Возвращаем уши в 0° (нейтральное положение)
service_ears(0, 0)

# Возвращаем шею в 0° и устанавливаем длительность движения 1 секунду
service_neck(0, 0, 1, 1)

print('Сценарий "Улыбнись" завершён. Все механизмы возвращены в исходное положение.')
```

---

## Шаг 4. Запуск сервисов и выполнение скрипта

1. **Остановка системного сервиса `robohead.service`.**  
  
   ```bash
   sudo systemctl stop robohead.service
   ```


2. **Запуск зависимостей (драйверов).**  
   
   ```bash
   roslaunch robohead_controller dependencies.launch
   ```
   Эта команда запустит все необходимые драйверы (*display_driver*, *ears_driver*, *neck_driver*, *speakers_driver*).

3. **Запуск нашего скрипта.**  
   В том же терминале, где расположен файл `main.py`, выполните:
   ```bash
   python3 ~/scripts/main.py
   ```
   Скрипт автоматически подключится к запущенным сервисам и выполнит описанный сценарий.

4. **Повторный запуск.**  
   После первого запуска скрипта можно многократно перезапускать его командой:
   ```bash
   python3 ~/scripts/main.py
   ```
   После перезагрузки Робоголовы, повторите шаг 1 (остановку сервиса), затем шаг 2 (запуск зависимостей).

---