## Вспомогательные вещи ##
Для просмотра характеристик подключенных камер используется **v4l2-ctl**:  
  >**v4l2-ctl --list-devices** (просмотр всех доступных устройств)  
  >**v4l2-ctl --device=/dev/video0 --list-formats-ext** (Просмотр информации о конкретном устройстве (разрешение и FPS))  
  >**v4l2-ctl -d0 --list-formats-ext -w** (Показывает больше информации чем предыдущая команда, (-d0 более кратко))  

Для проверки наличия плагина на данном устройстве используйте:  
  >**gst-inspect-1.0 | grep h264** (h264 кусок ключевого слова для примера)  

Для просмотра характеристик плагина:  
  >**gst-inspect-1.0 v4l2src** (v4l2src название плагина для примера, если он есть)

## Примитивное отображение информации ##
Для просмотра напрямую на экране (даже без графической оболочки Linux)  
 - Для первой камеры из стереопары  
>**DISPLAY=:0.0 gst-launch-1.0 nvarguscamerasrc sensor-id=0 ! 'video/x-raw(memory:NVMM), width=3280, height=2464, format=(string)NV12, framerate=(fraction)20/1' ! nvoverlaysink -e**  
 - Для второй камеры из стереопары  
>**DISPLAY=:0.0 gst-launch-1.0 nvarguscamerasrc sensor-id=1 ! 'video/x-raw(memory:NVMM), width=3280, height=2464, format=(string)NV12, framerate=(fraction)20/1' ! nvoverlaysink -e**

## Примеры пайплайнов для разных ситуаций: ##
В пайплайнах важно соблюдать последовательность плагинов. Для преобразование формата пикселей нужно использовать конвертеры **videoconvert** или **nvvidconv"** или другие, причем нужно указывать **capabilities** между ними и максимально подробно (причем обращать внимание на кавычки, если вдруг используются скобки), например:   
>**video/x-raw(memory:NVMM), width=640, height=480, format=(string)NV12, framerate=60/1**

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

### Данные "КАМЕРА -> КОД OpenCV"  
**flip-method=3** поворачивает картинку
>**VideoCapture cam0("nvarguscamerasrc sensor-id=0 ! video/x-raw(memory:NVMM), width=1280, height=720, format=(string)NV12, framerate=(fraction)30/1 ! nvvidconv flip-method=0 ! video/x-raw, width=1280, height=720, format=(string)BGRx, framerate=(fraction)30/1 ! videoconvert !video/x-raw, width=1280, height=720, format=(string)BGR, framerate=(fraction)30/1 ! appsink sync=false", CAP_GSTREAMER);**

### Данные "КОД OpenCV -> ВЫВОД НА ЭКРАН"
>**VideoWriter cam_out("appsrc ! video/x-raw, width=1280, height=720, format=(string)BGR, framerate=30/1 ! videoconvert ! video/x-raw, width=1280, height=720, format=(string)BGRx, framerate=30/1 ! nvvidconv flip-method=0 ! video/x-raw(memory:NVMM), width=1280, height=720, format=(string)NV12, framerate=(fraction)30/1 ! nvoverlaysink sync=false" , CAP_GSTREAMER, 30, Size(1280, 720), true);**


## Для дебага ##
Для использование дебагера gstreamer необходимо установить значение **GST_DEBUG**:  
Возможные значение:  
> - 0 | none    | No debug information is output.  
> - 1 | ERROR   | Logs all fatal errors. These are errors that do not allow the core or elements to perform the requested action. The application can still recover if programmed to handle the conditions that triggered the error.  
> - 2 | WARNING | Logs all warnings. Typically these are non-fatal, but user-visible problems are expected to happen.  
> - 3 | FIXME   | Logs all "fixme" messages. Those typically that a codepath that is known to be incomplete has been triggered. It may work in most cases, but may cause problems in specific instances.  
> - 4 | INFO    | Logs all informational messages. These are typically used for events in the system that only happen once, or are important and rare enough to be logged at this level.  
> - 5 | DEBUG   | Logs all debug messages. These are general debug messages for events that happen only a limited number of times during an object's lifetime; these include setup, teardown, change of parameters, etc.  
> - 6 | LOG     | Logs all log messages. These are messages for events that happen repeatedly during an object's lifetime; these include streaming and steady-state conditions. This is used for log messages that happen on every buffer in an element for example.  
> - 7 | TRACE   | Logs all trace messages. Those are message that happen very very often. This is for example is each time the reference count of a GstMiniObject, such as a GstBuffer or GstEvent, is modified.  
> - 8 | MEMDUMP | Logs all memory dump messages. This is the heaviest logging and may include dumping the content of blocks of memory.  

Устанавливается с помощью команды:
>**export GST_DEBUG="*:4"**  

## ТОНКОСТИ ##
 - использование плагинов с приставкой **nv** позволяет добиться большей производительности, поскольку вычисления происходят где-то в своей NVMM памяти  
 - Торможение картинки при отображении может быть связано с тем, что у используемого плагина: (я столкнулся при использовании **nvoverlaysink**) должно быть выставлено значение свойства **sync=false**, еще такое свойство есть у **appsink**.  
 - **nvvidconv** копирует из памяти NVMM в память процессора, если слева от него в конвейере **video/x-raw(memory:NVMM)**, а справа просто **video/x-raw**


