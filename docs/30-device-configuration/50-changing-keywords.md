---
id: 50-changing-keywords
title: Смена ключевой фразы
---

# Смена ключевой фразы

Распознаванием ключевых слов занимается ROS-нода [~kws_recognizer](../10-introduction/40-software/47-voice-recognizer-pocketsphinx.md#основные-возможности-ros-node-voice-pocketspinx). Далее будет приведен пример смены ключевой фразы по умолчанию `Слушай, Робот!` на `Эй, Голова!`

## Шаг 1. Остановка всех запущенных пакетов

Прежде чем вносить изменения, остановим текущий сервис, чтобы избежать неожиданных сбоев:

```bash
sudo systemctl stop robohead.service
```
> **Подсказка:** Вы можете проверить статус сервиса командой `sudo systemctl status robohead.service`. Если всё остановилось корректно, увидите статус `inactive`.

## Шаг 2. Изменение списка ключевых фраз

1. Откройте файл со списком фраз:

    ```
    robohead_ws/src/robohead/robohead_controller/config/voice_recognizer_pocketsphinx/kwslist.txt
    ```

2. Найдите строку с вашей старой фразой и замените её на новую:

    ```text
    эйголова /1e-40/
    ```

    - **Совет по порогу чувствительности (/1e-40/):**  
      - Меньше значение (например, `1e-45`) — робот будет менее «слуховым», распознаёт только чёткие команды.  
      - Большее значение (например, `1e-35`) — более чувствительный, но может реагировать на случайные звуки.

3. Обратите внимание: ключевую фразу нужно писать **без пробелов**, а сама фраза должна состоять из 3–4 слогов для надёжного распознавания (например, «Алиса», «Эй, Сири», «Окей, Гугл»).

---

## Шаг 3: Обновление фонетического словаря

Чтобы система правильно «понимала», как звучат слова, необходимо обновить фонетический словарь

Файл `robohead_controller/config/voice_recognizer_pocketsphinx/dictionary.txt` - список вообще всех слов, которые должен распознавать пакет.
В этот файл необходимо вписывать все слова (и ключевые слова, и слова, входящие в состав команд), которые должен распознавать пакет. 

1. Откройте `dictionary.txt`, он находится по пути:

    ```
    robohead_ws/src/robohead/robohead_controller/config/voice_recognizer_pocketsphinx/dictionary.txt
    ```
2. Вместо текущей ключевой фразы впишите новую ключевую фразу: 

    ```yaml
    # dictionary.txt
    эйголова
    покажи
    уши
    левое
    правое
    поздоровайся
    сделай
    фото
    следи
    за
    шариком
    ```
3. Сгенерируйте файл с фонетическими записями командой:

    ```bash
    ~/robohead_ws/src/ru4sphinx/text2dict/dict2transcript.pl \
      ~/robohead_ws/src/robohead/robohead_controller/config/voice_recognizer_pocketsphinx/dictionary.txt \
      ~/robohead_ws/src/robohead/robohead_controller/config/voice_recognizer_pocketsphinx/dictionary.dict
    ```

:::note
Первая часть команды: `~/robohead_ws/src/ru4sphinx/text2dict/dict2transcript.pl` - использование утилиты `ru4sphinx` для генерации фонетического словаря
    
Вторая часть команды: `~/robohead_ws/src/robohead/voice_recognizer_pocketsphinx/config/dictionary.txt` - это путь до файла, в котором лежит набор слов, которые необходимо распознавать.

Третья часть команды: `~/robohead_ws/src/robohead/voice_recognizer_pocketsphinx/config/dictionary.dict` - это путь, куда будет сохранён фонетический словарь (произошения слов).
:::
    
    - Утилита `ru4sphinx` сгенерирует два файла `dictionary.dict.accent` и `dictionary.dict`.
    - Файл `dictionary.dict.accent` содержит ударения в словах, по сути, это просто промежуточный файл для генерации `dictionary.dict`. Его дополнительное редактирование ни к чему не приведет.
    - **Совет:** откройте `dictionary.dict` и, если необходимо, добавьте альтернативные варианты произношения для сложных слов.

---

### Пример того, как могут выглядеть записи:

```yaml
# dictionary.dict
эйголова y j g ay l a v aa
эйголова(2) y j g o l o v aa
эйголова(3) ay j g o l a v aa
```

## Шаг 4: Настройка соответствий команд

Помимо того, что ключевую фразу нужно добавить в фонетический словарь, также нужно отразить, что именно по этой фразе будет происходить переход в стандартное действие `std_attention` для ожидания получения команды к выполнению.
Для этого необходимо отредактировать файл `robohead_ws/src/robohead/robohead_controller/config/robohead_controller.yaml`:

1. Откройте файл:

    ```
    robohead_ws/src/robohead/robohead_controller/config/robohead_controller.yaml
    ```

2. Найдите блок `robohead_controller_actions_match` и новую ключевую фразу `эйголова` для действия `std_attention`:

    ```yaml
    robohead_controller_actions_match:
      "эйголова": "robohead_controller_actions.std_attention.action"
      # остальные соответствия...
    ```

3. Полный пример блока для наглядности:

    ```yaml
    robohead_controller_actions_match:
      "wait_action": "robohead_controller_actions.std_wait.action"
      "low_bat_action": "robohead_controller_actions.std_low_bat.action"
      "эйголова": "robohead_controller_actions.std_attention.action"
      "покажи левое ухо": "robohead_controller_actions.std_left_ear.action"
      "покажи правое ухо": "robohead_controller_actions.std_right_ear.action"
      "покажи уши": "robohead_controller_actions.std_ears.action"
      "сделай фото": "robohead_controller_actions.std_make_photo.action"
      "поздоровайся": "robohead_controller_actions.std_greeting.action"
      "следи за шариком": "robohead_controller_actions.std_ball_tracker.action"
    ```
    
---

## Шаг 5. Перезапуск сервиса

После всех изменений перезапустите сервис, чтобы робот «подхватил» новую конфигурацию:

```bash
sudo systemctl restart robohead.service
```

> **Проверка:** снова используйте `sudo systemctl status robohead.service` и убедитесь, что статус — `active`.

---

## Часто возникающие вопросы и рекомендации

- **Робот не реагирует на «Эй, Голова!»**  
  - Проверьте, правильно ли указана фраза без пробелов.  
  - Попробуйте изменить порог чувствительности (секцию `/1e-40/`).  

- **Робот реагирует на неожиданные звуки**  
  - Увеличьте порог (например, до `1e-45`), чтобы отфильтровать фоновые шумы.

- **Фразы распознаются некорректно**  
  - Добавьте более подробные фонетические варианты в `dictionary.dict`.  

---