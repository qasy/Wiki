Для просмотра характеристик подключенных камер используется **v4l2-ctl**:  
  >**v4l2-ctl --list-devices** (просмотр всех доступных устройств)  
  >**v4l2-ctl --device=/dev/video0 --list-formats-ext** (Просмотр информации о конкретном устройстве (разрешение и FPS))  
  >**v4l2-ctl -d0 --list-formats-ext -w** (Показывает больше информации чем предыдущая команда, (-d0 более кратко))  

Для просмотра напрямую на экране (даже без графической оболочки Linux)  
 - Для первой камеры из стереопары  
>**DISPLAY=:0.0 gst-launch-1.0 nvarguscamerasrc sensor-id=0 ! 'video/x-raw(memory:NVMM), width=3280, height=2464, format=(string)NV12, framerate=(fraction)20/1' ! nvoverlaysink -e**  
 - Для второй камеры из стереопары  
>**DISPLAY=:0.0 gst-launch-1.0 nvarguscamerasrc sensor-id=1 ! 'video/x-raw(memory:NVMM), width=3280, height=2464, format=(string)NV12, framerate=(fraction)20/1' ! nvoverlaysink -e**

### Передача по UDP (на джетсоне)  
**sensor-id=0** указывает номер устройства /dev/video0  
**sensor-mode=2** указывает режим работы устройства (его разрешение и ФПС), который можно увидеть через **v4l2-ctl**  
**host=192.168.223.77 port=5000** указывает ip адрес и порт устройства, на которое отсылаем
>**gst-launch-1.0 nvarguscamerasrc sensor-id=0 sensor-mode=2 ! nvvidconv ! jpegenc ! rtpjpegpay ! udpsink host=192.168.223.77 port=5000**
  
Или так:  
(**quality** указывает степень сжатия для encoder)  
>**gst-launch-1.0 nvarguscamerasrc sensor-id=1 sensor-mode=3 ! nvvidconv ! jpegenc quality=20 ! rtpjpegpay ! udpsink host=192.168.223.77 port=5000** 

### Прием по UDP  
>**gst-launch-1.0 udpsrc port=5000 ! application/x-rtp, media=video, clock-rate=90000, encoding-name=JPEG, payload=96 ! rtpjpegdepay ! jpegdec ! videoconvert ! xvimagesink**

Какой-то Маринин пайплайн  
>**gst-launch-1.0 nvarguscamerasrc sensor-id=0 ! 'video/x-raw(memory:NVMM), width=640, height=480, format=(string)NV12, framerate=60/1' ! nvvidconv flip-method=3 ! 'video/x-raw(memory:NVMM), format=I420' ! omxh264enc qp-range=35,35:35,35:-1,-1 ! mpegtsmux ! udpsink clients=192.168.223.95:5001 sync=false**
  
(так вроде тоже работает)  
>**gst-launch-1.0 nvarguscamerasrc sensor-id=0 ! 'video/x-raw(memory:NVMM), width=640, height=480, format=(string)NV12, framerate=60/1' ! nvvidconv flip-method=3 ! 'video/x-raw(memory:NVMM), format=I420' ! omxh264enc ! udpsink clients=192.168.223.95:5001 sync=false**
