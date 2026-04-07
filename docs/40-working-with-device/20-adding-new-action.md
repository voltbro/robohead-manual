---
id: 20-adding-new-action
slug: adding-new-action
title: "Написание своего действия для robohead_controller"
sidebar_label: "Написание своего действия для robohead_controller"
sidebar_position: 2
description: "Написание своего действия для robohead_controller"
---

# Написание своего действия для robohead_controller

В этом разделе рассмотрим, как добавить собственное действие "**Улыбнись**", вызываемое голосовой командой, в систему управления Робоголовы (robohead_controller). Процесс состоит из трёх этапов:

1. **Добавление новой голосовой команды в словарь распознавания речи**  
2. **Создание скрипта действия**  
3. **Настройка конфигурации пакета robohead_controller**

![Собака-улыбака](attachments/smile.png)

Каждый шаг подробно описан ниже.

---

## 1. Добавление новой голосовой команды в словарь распознавания речи

Для того чтобы Робоголова могла распознавать команду «улыбнись», нужно обновить формальную грамматику и фонетический словарь движка PocketSphinx.

### 1.1 Обновление файла грамматики (gram.txt)

Формальная грамматика определяет множество команд, которые может распознавать речевой модуль. Откройте файл:

```bash
nano ~robohead_ws/src/robohead/robohead_controller/config/voice_recognizer_pocketsphinx/gram.txt
```

В нём вы увидите примерно такой набор правил:

```jsgf
#JSGF V1.0;

grammar robohead_cmds;

public <commands> = <command> ;

<command> = <command_1> | <command_2> | <command_3> | <command_4>;

<command_1> = покажи ( уши | левое ухо | правое ухо ) ;
<command_2> = поздоровайся ;
<command_3> = сделай фото ;
<command_4> = следи за шариком ;
```

Чтобы добавить новую команду `<command_5> = улыбнись`, нужно:

1. Расширить список команд, включив `<command_5>` в правило `<command>`.
2. Добавить определение `<command_5>`.

В результате ваш файл должен выглядеть так:

```jsgf
#JSGF V1.0;

grammar robohead_cmds;

public <commands> = <command> ;

<command> = 
    <command_1> 
  | <command_2> 
  | <command_3> 
  | <command_4> 
  | <command_5> ;

<command_1> = покажи ( уши | левое ухо | правое ухо ) ;
<command_2> = поздоровайся ;
<command_3> = сделай фото ;
<command_4> = следи за шариком ;
<command_5> = улыбнись ;
```

> **Примечание:** Формат JSGF чувствителен к синтаксису: убедитесь, что конструкции `|` и `;` стоят на своих местах, и нет лишних пробелов или символов.

### 1.2 Обновление фонетического словаря слов (dictionary.dict)

Для автоматической генерации фонетического словаря будем использовать специальную утилиту, но перед этим нам нужно писать **все** слова, из которых будет составлен фонетический словарь: в нем должны быть перечислены все ключевые слова, и составляющие команд (слова, предлоги, союзы и т.д.)

Откройте файл, где будут перечислены все слова, из которых будет строиться фонетический словарь:

```bash
nano ~robohead_ws/src/robohead/robohead_controller/config/voice_recognizer_pocketsphinx/dictionary.txt
```

Добавьте слово `улыбнись` в конец списка (или в алфавитном порядке для удобства):

```
слушайробот
покажи
уши
ухо
левое
правое
поздоровайся
сделай
фото
следи
за
шариком
улыбнись
```

Теперь сконвертируем этот текстовый словарь `dictionary.txt` в фонетический словарь `dictionary.dict`. Используем для этого утилиту `dict2transcript.pl`, расположенную в пакете `ru4sphinx`.

Запустите команду:

```bash
~/robohead_ws/src/ru4sphinx/text2dict/dict2transcript.pl   ~/robohead_ws/src/robohead/robohead_controller/config/voice_recognizer_pocketsphinx/dictionary.txt   ~/robohead_ws/src/robohead/robohead_controller/config/voice_recognizer_pocketsphinx/dictionary.dict
```

В результате этого скрипта будет создан файл `dictionary.dict`, содержащий фонетические транскрипции слов. Примерный результат:

```
за z aa
левое ll je v ay i
поздоровайся p ay z d a r oo v ay j ss i
покажи p ay k a zh yy
правое p r aa v ay i
сделай z dd je l ay j
следи s ll i dd ii
слушайробот s l uu sh ay j r ay b ay t
улыбнись u l y b nn ii ss
ухо uu h ay
уши uu sh y
фото f oo t ay
шариком sh aa rr i k ay m
```

> **Важно:** В некоторых случаях произношение одного слова может зависеть от контекста или ударения. Чтобы улучшить распознавание, можно добавить альтернативные варианты транскрипции, например:

```
следи s ll i dd ii
следи(2) s ll je dd ii

левое ll je v ay i
левое(2) ll je v oo je

улыбнись u l y b nn ii ss
```

Каждую альтернативную транскрипцию можно поместить в отдельную строку, указав слово с суффиксом `(2)` — это поможет PocketSphinx распознавать команды с разным тембром и ударением в речи.

Базовый набор альтернативных транскрипций можете посмотреть ДО конвертации в файле `dictionary.dict`, после конвертации они будут перезаписаны. 

---

## 2. Создание скрипта действия

Теперь, когда новая команда распознаётся речевым движком, необходимо написать сам код, который будет выполняться при активации «улыбнись». Для этого нам также понадобятся 2 файла:

[Картинка, выводимая на экран. smile.png](attachments/smile.png)

[Аудио-файл, воспроизводимый через динамики. smile.mp3](attachments/smile.mp3)

### 2.1 Структура папки скрипта и медиа-файлов

Перейдите в директорию, где хранятся скрипты действий:

```bash
cd ~/robohead_ws/src/robohead/robohead_controller/scripts/robohead_controller_actions
```

Создайте папку `smile` для нового действия
```bash
mkdir smile
```

В папке `smile` создайте файл `action.py`. Переместите в эту же папку два медиа-файла: изображение улыбки (`smile.png`) и звуковой файл с «улыбкой» (`smile.mp3`). Если вы используете удалённое подключение (SFTP или VSCode Remote), просто перетащите эти файлы в папку `smile`.

![Состав папки `smile`](attachments/smile-action.png)

### 2.2 Написание основного кода (action.py)

Откройте файл `action.py` и вставьте следующий код, который реализует действие «улыбки»:

```python
from robohead_controller_actions.main import *

def run(robohead_controller: RoboheadController, cmds: str):
    # Получаем путь к текущему скрипту (чтобы корректно ссылаться на медиа-файлы)
    script_path = os.path.dirname(os.path.abspath(__file__)) + '/'

    # 1) Вывод изображения 'smile.png' на экран робота
    msg_img = PlayMediaRequest()
    msg_img.is_blocking = 0        # не блокировать основной поток
    msg_img.is_cycled = 0          # показывать изображение один раз
    msg_img.path_to_file = script_path + 'smile.png'
    robohead_controller.display_driver_srv_PlayMedia(msg_img)

    # 2) Поворот шеи к источнику звука
    msg_neck = NeckSetAngleRequest()
    msg_neck.horizontal_angle = min(max(robohead_controller.respeaker_driver_doa_angle, -30), 30)
    msg_neck.vertical_angle = 30
    msg_neck.duration = 1          # время движения в секундах
    msg_neck.is_blocking = 0       # не ждать окончания движения шеи
    robohead_controller.neck_driver_srv_NeckSetAngle(msg_neck)

    # 3) Разворот ушей в стороны для эффекта «улыбки»
    msg_ears = EarsSetAngleRequest()
    msg_ears.left_ear_angle = -90   # левое ухо влево
    msg_ears.right_ear_angle = 90   # правое ухо вправо
    robohead_controller.ears_driver_srv_EarsSetAngle(msg_ears)

    # 4) Проигрывание звукового файла 'smile.mp3'
    msg_audio = PlayAudioRequest()
    msg_audio.path_to_file = script_path + 'smile.mp3'
    msg_audio.is_blocking = 1      # дождаться завершения воспроизведения
    msg_audio.is_cycled = 0        # воспроизвести один раз
    robohead_controller.speakers_driver_srv_PlayAudio(msg_audio)
```

**Что происходит в коде:**

1. **PlayMediaRequest:** отправляет на экран робота изображение `smile.png`, создавая визуальный эффект «улыбки».  
2. **NeckSetAngleRequest:** поворачивает голову к говорящему, используя угол `respeaker_driver_doa_angle`, возвращаемый микрофоном ReSpeaker. Ограничение угла в диапазоне от −30° до +30° обеспечивает плавность и безопасность движения.  
3. **EarsSetAngleRequest:** разворачивает уши в стороны, устанавливая углы на −90° (левое ухо) и +90° (правое ухо), что усиливает впечатление «улыбающегося» робота.  
4. **PlayAudioRequest:** приостанавливает дальнейшие действия до окончания воспроизведения аудиофайла `smile.mp3`, чтобы завершить «улыбку» звуковым сопровождением.

---

## 3. Настройка конфигурации пакета robohead_controller

Осталось связать голосовую команду с написанным скриптом. Для этого редактируем файл `robohead_controller.yaml`.

### 3.1 Открытие конфигурационного файла

Файл находится по пути:

```
~robohead_ws/src/robohead/robohead_controller/config/robohead_controller.yaml
```

Откройте его в любом удобном редакторе.

### 3.2 Добавление новой команды в словарь `robohead_controller_actions_match`

Найдите секцию, отвечающую за сопоставление распознанных команд с соответствующими скриптами:

```yaml
robohead_controller_actions_match: {
  "wait_action": "robohead_controller_actions.std_wait.action",
  "low_bat_action": "robohead_controller_actions.std_low_bat.action",
  "слушайробот": "robohead_controller_actions.std_attention.action",
  "покажи левое ухо": "robohead_controller_actions.std_left_ear.action",
  "покажи правое ухо": "robohead_controller_actions.std_right_ear.action",
  "покажи уши": "robohead_controller_actions.std_ears.action",
  "сделай фото": "robohead_controller_actions.std_make_photo.action",
  "поздоровайся": "robohead_controller_actions.std_greeting.action",
  "следи за шариком": "robohead_controller_actions.std_ball_tracker.action",
  # Добавляем связку для команды «улыбнись»
  "улыбнись": "robohead_controller_actions.smile.action",
}
```

> **Совет:** Следите за правильными кавычками и отсутствием лишних запятых, чтобы YAML-файл оставался рабочим.

## 4. Применение изменений

Для применения изменений в конфигурационных файлах (движка распознавания речи и `robohead_controller.yaml`) требуется перезапуск пакета `robohead_controller`. Сделать это можно тремя способами:
- Остановить Ubuntu-сервис и запустить пакет в ручном режиме для отладки (наиболее предпочтительный вариант, так как можно сразу увидеть появляющиеся ошибки)
- Перезапустить Ubuntu-сервис (удобен для проверки работы того, как будет работать Робоголова после перезагрузки питания)
- Физически перезагрузить Робоголову через кнопку питания (наименее удобный вариант, посколько придется ждать загрузки всей системы заново)

Далее проделайте **один** из шагов 4.1 или 4.2, или перезагрузите голову.
### 4.1 Запуск в режиме отладки

Остановите Ubuntu-сервис (в этот момент пакет `robohead_controller` и все зависимости автоматически остановятся)
```bash
# Останавливаем сервис robohead
sudo systemctl stop robohead.service
```
Запустите `robohead_controller` и все зависимости в ручном режиме:

```bash
roslaunch robohead_controller robohead_controller_py.launch
```
На экране терминала должен появиться примерно следующий вывод:

![Запуск без ошибок](attachments/robohead-launch.png)

### 4.2 Перезапуск Ubuntu-сервиса

```bash
# Останавливаем сервис robohead, если не был остановлен
sudo systemctl stop robohead.service

# Запускаем сервис заново
sudo systemctl start robohead.service
```

Для удобства вместо этих двух комманд можно использовать:
```bash
sudo systemctl restart robohead.service
```
Если всё прошло успешно, новый скрипт будет загружен, и Робоголова сможет реагировать на команду «улыбнись».

## 5. Итоговое поведение

После выполнения всех вышеописанных шагов, при произнесении команды **«Слушай, Робот! Улыбнись!»** произойдёт следующее:

1. Голова робота определит направление голоса и плавно повернётся к говорящему (±30° по горизонтали, +30° по вертикали).  
2. Уши робота развернутся в стороны (левое ухо на −90°, правое на +90°), создавая эффект «улыбки».  
3. На экран выведется изображение `smile.png`, усиливая визуальное восприятие «улыбки».  
4. Проиграется аудиофайл `smile.mp3`, добавляя звук к «улыбающемуся» роботу.  

Таким образом, вы получите полностью работающее и наглядное голосовое действие, которое легко редактировать и расширять в дальнейшем.

## 6. Отладка команд
В этом шаге описано как запускать стандартные действия, не произнося каждый раз голосом нужную команду.

Отлаживать команды можно как при автоматически запущенном `robohead_controller` (через Ubuntu-сервис), так и когда он запущен в ручном режиме (этот вариант предпочтительнее для отладки, так как видны все возникающие ошибки)

Откройте новое окно терминала, подключитесь к Робоголове и опубликуйте "**улыбнись**" в топик `/robohead_controller/voice_recognizer_pocketsphinx/cmds_recognizer/commands`

```bash
# Запустите в отдельном терминале
rostopic pub /robohead_controller/voice_recognizer_pocketsphinx/cmds_recognizer/commands std_msgs/String "data: 'улыбнись'"
```

После публикации команды Робоголова должна выполнить ваше действие.

В топик нужно отправлять именно ту строку символов, которая была указана в `robohead_controller_actions_match` файла `robohead_controller.yaml`