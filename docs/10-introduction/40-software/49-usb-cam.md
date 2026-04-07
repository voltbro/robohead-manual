---
id: 49-usb-cam
slug: usb_cam
title: "usb_cam"
sidebar_label: "usb_cam"
description: "usb_cam"
draft: true
---

# Пакет `usb_cam`

**Назначение:** Пакет `usb_cam` обеспечивает захват видеопотока с USB-камеры и публикацию изображений в формате ROS Image сообщений.

---

## Основные возможности

1. **Публикация видеокадров**

   * **Топик:** `/robohead_controller/usb_cam/image_raw`
   * **Сообщение:** `sensor_msgs/Image`
   * Публикация в режиме реального времени с заданной частотой (по умолчанию 30 FPS).

2. **Публикация информации о камере**

   * **Топик:** `/robohead_controller/usb_cam/camera_info`
   * **Сообщение:** `sensor_msgs/CameraInfo`
   * Содержит параметры калибровки и характеристики изображения.

3. **Настраиваемые параметры**

   * Файл: `config/usb_cam.yml`
   * Основные настройки:

     | Параметр       | Описание                                       |
     | -------------- | ---------------------------------------------- |
     | `video_device` | Путь к устройству (например, `/dev/video0`)    |
     | `image_width`  | Ширина изображения (px)                        |
     | `image_height` | Высота изображения (px)                        |
     | `pixel_format` | Формат пикселя (например, `mjpeg`, `yuyv`)     |
     | `frame_rate`   | Частота кадров (FPS)                           |
     | `io_method`    | Метод ввода-вывода (`mmap`, `read`, `userptr`) |

---

*См. подробности на официальном портале ROS: [wiki.ros.org/usb\_cam](https://wiki.ros.org/usb_cam) и репозиторий [ros-drivers/usb\_cam](https://github.com/ros-drivers/usb_cam).*
