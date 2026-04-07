---
id: 35-microphone-backlight-control
slug: microphone-backlight-control
title: "Управление подсветкой микрофонного модуля"
sidebar_label: "Управление подсветкой микрофонного модуля"
sidebar_position: 6
description: "Управление подсветкой микрофонного модуля"
---

# Управление подсветкой микрофонного модуля

Модуль ReSpeaker USB Mic Array оснащён кольцом из RGB-светодиодов (Pixel Ring). Подсветку можно настроить через ROS-сервис и параметры конфигурации.

---

## Конфигурация подсветки

Основные настройки находятся в конфигурационном файле `respeaker_driver.yaml`:

```yaml
# ~/robohead_ws/src/robohead/robohead_controller/config/respeaker_driver.yaml

led:
  brightness: 10 # 0..31 яркость светодиодной панели
  A_color: # цвет светодиода, указывающего направление свечения
    - 255 # R-red
    - 0 # G-green
    - 0 # B-blue
  B_color: # цвето светодиодов заливки
    - 255 # R-red
    - 255 # G-green
    - 255 # B-blue
  mode: 1
# modes: 
# 0 - off
# 1 - trace
# 2 - listen
# 3 - wait
# 4 - speak
# 5 - spin
```

> *Примечание:* яркость регулируется значением 0 (минимум) … 31 (максимум), цвета задаются списком из трёх байт (RGB). Сайт на котором можно по палитре получить значения R, G и B [->](https://www.rapidtables.com/web/color/RGB_Color.html)

---

## ROS-сервисы для управления светодиодами

Пакет `respeaker_driver` предоставляет несколько сервисов и топиков для тонкой настройки подсветки микрофонного модуля (изменения сохраняются до перезапуска пакета):

### 1. Сервис SetBrightnessLED

* **Сервис:** `robohead_controller/respeaker_driver/SetBrightnessLED`
* **Описание:** устанавливает яркость всей подсветки: (0–31).
* **Пример:**

  ```bash
  rosservice call /robohead_controller/respeaker_driver/SetBrightnessLED "brightness: 15"
  ```

### 2. SetColorPaletteLED

* **Сервис:** `robohead_controller/respeaker_driver/SetColorPaletteLED`
* **Описание:** задаёт два цвета:

  * `colorA` — светодиод, следящий за направленим звука;
  * `colorB` — заливка остальных светодиодов подсветки.
* **Параметры:** RGB-списки `[0–255,0–255,0–255]`.
* **Пример:**

  ```bash
  rosservice call /robohead_controller/respeaker_driver/SetColorPaletteLED "colorA:
  - 255
  - 0
  - 0
  colorB:
  - 0
  - 123
  - 255"
  ```

### 3. SetModeLED

* **Сервис:** `robohead_controller/respeaker_driver/SetModeLED`
* **Описание:** переключает режим работы светодиодного кольца. Ниже перечислены все режимы:

| Код | Режим  | Поведение и параметры                                                                                |
| --- | ------ | ---------------------------------------------------------------------------------------------------- |
| 0   | off    | Полное отключение подсветки.                                                                         |
| 1   | trace  | Режим «слежения»: заливается цветом `B_color`, а `A_color` указывает направление звука. |
| 2   | listen | Режим «ожидания»: постоянная подсветка всего кольца цветами `A_color` (указательный LED) и `B_color` (фон).       |
| 3   | wait   | Переключение между `A_color` и `B_color` через заданные интервалы (мигание). Не следит за направлением звука!                        |
| 4   | speak  | Чередование: эффекты «бегущей волны» цветами `A_color`/`B_color`. Не следит за направлением звука!     |
| 5   | spin   | Анимация: постепенная заливка кольца цветом `A_color` с эффектом затухания предыдущих светодиодов. Не следит за направлением звука!    |

* **Пример вызова:**

  ```bash
  rosservice call /robohead_controller/respeaker_driver/SetModeLED "mode: 1"
  ```

### 4. Сервис SetColorAllLED

* **Сервис:** `robohead_controller/respeaker_driver/SetColorAllLED`
* **Описание:** заливает все светодиоды одним цветом.
* **Параметры:** три отдельных канала `r`, `g`, `b` (0–255).
* **Пример:**

  ```bash
  rosservice call /robohead_controller/respeaker_driver/SetColorAllLED "r: 254
  g: 127
  b: 64"
  ```

### 5. Топик SetColorManualLED

* **Топик:** `robohead_controller/respeaker_driver/SetColorManualLED`

* **Тип сообщения:** `std_msgs/ColorRGBA`

* **Описание:** позволяет задать индивидуальный цвет для каждого из 12 светодиодов кольца. Цвета задаются массивом из 12 сообщений типа `std_msgs/ColorRGBA` значений RGB.

* **Пример:**

```bash
 rostopic pub /robohead_controller/respeaker_driver/SetColorManualLED respeaker_driver/SetColorManualLED "colors:
- {r: 0.0, g: 0.0, b: 255.0, a: 0.0}
- {r: 0.0, g: 255.0, b: 0.0, a: 0.0}
- {r: 255.0, g: 0.0, b: 0.0, a: 0.0}
- {r: 0.0, g: 0.0, b: 255.0, a: 0.0}
- {r: 0.0, g: 255.0, b: 0.0, a: 0.0}
- {r: 255.0, g: 0.0, b: 0.0, a: 0.0}
- {r: 0.0, g: 0.0, b: 255.0, a: 0.0}
- {r: 0.0, g: 255.0, b: 0.0, a: 0.0}
- {r: 255.0, g: 0.0, b: 0.0, a: 0.0}
- {r: 0.0, g: 0.0, b: 255.0, a: 0.0}
- {r: 0.0, g: 255.0, b: 0.0, a: 0.0}
- {r: 255.0, g: 0.0, b: 0.0, a: 0.0}"
```

---

## Примечания

* Остальные параметры конфигурационного файла (USB, audio, ROS-топики) отвечают за инициализацию устройства и поток аудио и не влияют на подсветку.
* Настройки `led.A_color` и `led.B_color` из конфигурационного файла используются как значения по умолчанию при старте Робоголовы.
* После изменения конфигурации перезапустите сервис:

  ```bash
  sudo systemctl restart robohead.service
  ```



