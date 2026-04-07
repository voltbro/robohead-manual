---
id: 33-adjusting-volume
slug: adjusting-volume
title: "Настройка громкости динамиков"
sidebar_label: "Настройка громкости динамиков"
sidebar_position: 4
description: "Настройка громкости динамиков"
---

# 1. Общая информация

Громкостью динамиков на Робоголове управляет ROS-пакет [**`speakers_driver`**](../../10-introduction/40-software/45-speakers-driver.md). Этот пакет обеспечивает вывод звука на внешние динамики: воспроизведение аудио и установку громкости. 

Стартовые параметры динамиков настраиваются через управляющий пакет **`robohead_controller`**. Для изменения настроек потребуется доступ к конфигурационным файлам.

:::warning
**Важно!** Перед редактированием файлов остановите Linux-сервис:
```bash
sudo systemctl stop robohead.service
```
:::



# 2. Конфигурационный файл

Основные настройки находятся в конфигурационном файле `~/robohead_ws/src/robohead/robohead_controller/config/speakers_driver.yaml`:

```yaml
# ~/robohead_ws/src/robohead/robohead_controller/config/speakers_driver.yaml

service_PlayAudio_name: "~PlayAudio"      # Имя ROS-сервиса для воспроизведения аудиофайлов
service_GetVolume_name: "~GetVolume"      # Имя ROS-сервиса для проверки уровня громкости
service_SetVolume_name: "~SetVolume"      # Имя ROS-сервиса для установки уровня громкости
mpd_host: "/run/mpd/socket"               # Адрес MPD-сервера
mpd_port: 6600                            # Порт MPD-сервера
update_hz: 10                             # Частота обновления (Гц)
default_volume: 50                        # Громкость при запуске (0–100)
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
    nano ~/robohead_ws/src/robohead/robohead_controller/config/speakers_driver.yaml
    ```
4. Измените параметр `default_volume` (допустимые значения — от 0 до 100).



5. Сохраните изменения сочетанием клавиш <kbd>Ctrl+S</kbd> и выйдите из редактора <kbd>Ctrl+X</kbd>.
6. Запустите сервис:
    ```bash
    sudo systemctl start robohead.service
    ```
7. Проверьте текущую громкость:
    ```bash
    rosservice call robohead_controller/speakers_driver/GetVolume "{}"
    ```

## 3.2 Изменение громкости для текущего сеанса

> *Обратите внимание!*  Изменение громкости этим методом изменит ее только для текущего сеанса. После перезагрузки громкость будет установлена в значение по умолчанию из конфигурационного файла

В терминале Робоголовы выполните команду:

```bash
rosservice call robohead_controller/speakers_driver/SetVolume "volume: 30"
```

- **volume**: значение от 0 (звук отключён) до 100 (максимальная громкость)  
- Возвращает:  
  - `0` — успех  
  - `-1` — ошибка (некорректное значение)

# 4. Рекомендации

- Для учебных целей: громкость **30–50%**  
- Для шумных помещений: **60–80%**  
- Избегайте **100%** для предотвращения искажений
