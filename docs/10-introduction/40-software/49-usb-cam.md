---
id: 49-usb-cam
slug: usb_cam
title: "usb_cam"
sidebar_label: "usb_cam"
description: "usb_cam"
draft: false
---

# Пакет `usb_cam`

**Назначение:** Пакет `usb_cam` обеспечивает захват видеопотока с USB-камеры и публикацию изображений в формате ROS Image сообщений.

---

## Основные возможности

1. **Публикация видеокадров**

   * **Топик:** `image_raw`
   * **Сообщение:** `sensor_msgs/Image`
   * Публикация в режиме реального времени с заданной частотой (по умолчанию 30 FPS).

2. **Публикация информации о камере**

   * **Топик:** `camera_info`
   * **Сообщение:** `sensor_msgs/CameraInfo`
   * Содержит параметры калибровки и характеристики изображения.


## Запуск

```bash
ros2 run usb_cam usb_cam_node_exe
```

## Работа с `robohead_controller`

При запуске `robohead_controller` пакет `usb_cam` запускается автоматически launch-файлом `robohead_controller/launch/dependencies.launch.py`.

Параметры заупуска:
```python
Node(
   package='usb_cam',
   executable='usb_cam_node_exe',
   name='usb_cam',
   output='screen',
   namespace='usb_cam',
   parameters=[{
            'video_device': '/dev/video0',
            'image_width': 640,
            'image_height': 480,
            'framerate': 30.0,
            'pixel_format': 'mjpeg2rgb',
            'io_method': 'mmap',
            'frame_id': 'front_camera',
            'camera_info_url': '',
   }]
)
```

---

*См. подробности на официальном портале ROS2 Jazzy: [docs.ros.org/en/jazzy/p/usb_cam](https://docs.ros.org/en/jazzy/p/usb_cam/) и репозиторий [ros-drivers/usb\_cam](https://github.com/ros-drivers/usb_cam).*
