---
id: 33-adjusting-volume
slug: adjusting-volume
title: "Настройка громкости динамиков"
sidebar_label: "Настройка громкости динамиков"
sidebar_position: 4
description: "Настройка громкости динамиков"
---

# 1. Общая информация

Громкостью динамиков на Робоголове управляет ROS-пакет [**`media_driver`**](../../10-introduction/40-software/42-media-driver.md). Этот пакет обеспечивает вывод звука на внешние динамики: воспроизведение аудио и установку громкости. 

Стартовые параметры динамиков настраиваются через управляющий пакет **`robohead_controller`**. Для изменения настроек потребуется доступ к конфигурационным файлам.

:::warning
**Важно!** Перед редактированием файлов остановите Linux-сервис:
```bash
sudo systemctl stop robohead.service
```
:::



# 2. Конфигурационный файл

Основные настройки находятся в конфигурационном файле `~/robohead_ws/src/robohead/robohead_controller/config/media_driver.yaml`:

```yaml
# ~/robohead_ws/src/robohead2/robohead_controller/config/media_driver.yaml
/**:
  ros__parameters:

    srv_set_volume_name: "set_volume"           # Имя ROS-сервиса для установки уровня громкости
    srv_get_volume_name: "get_volume"           # Имя ROS-сервиса для проверки уровня громкости
    srv_play_media_name: "play_media"           # Имя ROS-сервиса для воспроизведения видео/аудио/картинок
    srv_is_idle_audio_name: "is_idle/audio"     # Имя ROS-сервиса для проверки ожидания воспроизведения аудио
    srv_is_idle_display_name: "is_idle/display" # Имя ROS-сервиса для проверки ожидания воспроизведения видео/картинок


    topic_stream_name: "stream"                 # Имя ROS-топика для потокового вывода изображения
    stop_command: "__STOP__"                    # Команда для остановки воспроизведения (видео либо звука)

    display_rotate: "270" # Поворот картинки дисплея (по часовой стрелке), 0-359 градусов
    default_volume: 60.0  # Громкость по-умолчанию при запуске

    # touchscreen config:
    device_name: "waveshare"                # Имя тачскрина
    device_path: "/dev/input/"              # Путь для поиска тачскрина
    topic_touchscreen_name: "touchscreen"   # Имя ROS-топика, куда публикуются касания тачскрина
```

# 3. Изменение громкости

## 3.1 Настройка громкости по умолчанию

1. Подключитесь к устройству по SSH:
    ```bash
    ssh pi@roboheadXXX.local
    ```
2. Остановите Linux-сервис:
    ```bash
    sudo systemctl stop robohead.service
    ```
3. Откройте файл настроек в текстовом редакторе:
    ```bash
    nano ~/robohead_ws/src/robohead2/robohead_controller/config/media_driver.yaml
    ```
4. Измените параметр `default_volume` (допустимые значения — от 0 до 100).



5. Сохраните изменения сочетанием клавиш <kbd>Ctrl+S</kbd> и выйдите из редактора <kbd>Ctrl+X</kbd>.
6. Запустите сервис:
    ```bash
    sudo systemctl start robohead.service
    ```
7. Проверьте текущую громкость:
    ```bash
     ros2 service call /robohead/media_driver/get_volume robohead_interfaces/srv/SimpleCommand "data: 0"
    ```

## 3.2 Изменение громкости для текущего сеанса

> *Обратите внимание!*  Изменение громкости этим методом изменит ее только для текущего сеанса. После перезагрузки громкость будет установлена в значение по умолчанию из конфигурационного файла

В терминале Робоголовы выполните команду:

```bash
ros2 service call /robohead/media_driver/set_volume robohead_interfaces/srv/SimpleCommand "data: 60"
```

- **volume**: значение от 0 (звук отключён) до 100 (максимальная громкость)  
- Возвращает:  
  - `0` — успех  
  - `-1` — ошибка (некорректное значение)

# 4. Рекомендации

- Для учебных целей: громкость **30–50%**  
- Для шумных помещений: **60–80%**  
- Избегайте **100%** для предотвращения искажений
