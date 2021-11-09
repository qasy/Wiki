## Вспомогательные вещи ##
Accelerated GStreamer (какие-то примеры пайплайнов от nvidia)
>https://docs.nvidia.com/jetson/l4t/index.html#page/Tegra%20Linux%20Driver%20Package%20Development%20Guide/accelerated_gstreamer.html#  

Еще пайплайны  
>https://gist.github.com/hum4n0id/cda96fb07a34300cdb2c0e314c14df0a  
>https://github.com/mad4ms/python-opencv-gstreamer-examples

Пайплайны для gstreamer на джетсон нано  
>https://developer.ridgerun.com/wiki/index.php?title=Category:JetsonNano



Для просмотра характеристик подключенных камер используется **v4l2-ctl**:  
  >**v4l2-ctl --list-devices** (просмотр всех доступных устройств)  
  >**v4l2-ctl --device=/dev/video0 --list-formats-ext** (Просмотр информации о конкретном устройстве (разрешение и FPS))  
  >**v4l2-ctl -d0 --list-formats-ext -w** (Показывает больше информации чем предыдущая команда, (-d0 более кратко))  

Для проверки наличия плагина на данном устройстве используйте:  
  >**gst-inspect-1.0 | grep h264** (h264 кусок ключевого слова для примера)  

Для просмотра характеристик плагина, его входов и выходов и других полезных параметров:  
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

### К следующему пункту, запихивает из камеры в код:  
>**"nvarguscamerasrc sensor-id=0 ! video/x-raw(memory:NVMM), width=1280, height=720, format=(string)NV12, framerate=60/1 ! nvvidconv flip-method=0 ! video/x-raw, width=1280, height=720, format=(string)BGRx, framerate=60/1 ! videoconvert ! video/x-raw, width=1280, height=720, format=(string)BGR, framerate=60/1 ! appsink sync=false"**

### Передача из кода OpenCV -> UDP sink ###
>**"appsrc ! video/x-raw, width=1280, height=720, format=(string)BGR ! videoconvert ! video/x-raw, format=(string)BGRx ! nvvidconv flip-method=0 ! nvjpegenc quality=20 ! rtpjpegpay ! udpsink host=192.168.223.77 port=5000"**  

### Прием из UDP src -> на экран монитора ноутбука###  
>**gst-launch-1.0 udpsrc port=5000 ! application/x-rtp, media=video, clock-rate=90000, encoding-name=JPEG, payload=96 ! rtpjpegdepay ! jpegdec ! videoconvert ! xvimagesink**

### Отправка на экран и по UDP с энкодером H264 
>gst-launch-1.0 nvarguscamerasrc ! \  
'video/x-raw(memory:NVMM),width=1280, height=720, framerate=30/1, format=NV12' ! \  
tee name=t \  
t. ! queue ! nvoverlaysink sync=false \  
t. ! queue ! omxh264enc insert-sps-pps=true bitrate=16000000 ! \  
rtph264pay ! \  
udpsink port=5000 host=192.168.223.77  

### Отправка из кода OpenCV по UDP с энкодером H264  
>std::string gst_writer = "appsrc ! \  
                                videoconvert ! \  
                                nvvidconv flip-method=0 ! \  
                                video/x-raw(memory:NVMM), width=2560, height=720 ! \  
                                tee name=t \  
                                t. ! queue ! \  
                                nvoverlaysink sync=false \  
                                t. ! queue ! \  
                                nvv4l2h264enc insert-sps-pps=true ! \  
                                h264parse ! \  
                                rtph264pay pt=96 ! \  
                                udpsink host=192.168.223.77 port=5000 sync=false -e";  
>VideoWriter cam_out(gst_writer, CAP_GSTREAMER, 30, Size(2560, 720), true); 

### Объединение двух камер через **nvcompositor** и передача в код OpenCV ###
>std::string gst_capture = "nvarguscamerasrc sensor-id=0 ! \
                                video/x-raw(memory:NVMM),width=1280, height=720, framerate=30/1, format=NV12 ! \  
                                comp. \  
                                nvarguscamerasrc sensor-id=1 ! \  
                                video/x-raw(memory:NVMM),width=1280, height=720, framerate=30/1, format=NV12 ! \  
                                comp. \  
                                nvcompositor name=comp \  
                                sink_0::xpos=0 sink_0::ypos=0 sink_0::width=1280 sink_0::height=720 \  
                                sink_1::xpos=1280 sink_1::ypos=0 sink_1::width=1280 sink_1::height=720 ! \  
                                video/x-raw(memory:NVMM) ! \  
                                nvvidconv ! \  
                                video/x-raw,format=BGRx ! \  
                                videoconvert ! \  
                                video/x-raw,format=BGR ! \  
                                appsink";  
>VideoCapture cam(gst_capture, CAP_GSTREAMER);  

### Объединение двух камер через **nvcompositor**, H264 и отправка по UDP ###
>nvarguscamerasrc sensor-id=0 ! \  
'video/x-raw(memory:NVMM),width=1280, height=720, framerate=30/1, format=NV12' ! \  
comp. \  
nvarguscamerasrc sensor-id=1 ! \  
'video/x-raw(memory:NVMM),width=1280, height=720, framerate=30/1, format=NV12' ! \  
comp. \  
nvcompositor name=comp \  
sink_0::xpos=0 sink_0::ypos=0 sink_0::width=1280 sink_0::height=720 \  
sink_1::xpos=1280 sink_1::ypos=0 sink_1::width=1280 sink_1::height=720 ! \  
'video/x-raw(memory:NVMM),width=2560, height=720' ! \  
nvvidconv ! \  
omxh264enc insert-sps-pps=true bitrate=16000000 ! \  
rtph264pay ! \  
udpsink port=5000 host=192.168.223.77  

### Прием по UDP на ноутбуке закодированного H264 двойного изображения с Jetson'a
gst-launch-1.0 -v udpsrc port=5000 ! \  
"application/x-rtp, media=(string)video, clock-rate=(int)90000, encoding-name=(string)H264, payload=(int)96" ! \
rtph264depay ! \
h264parse ! \
avdec_h264 ! \
videoconvert ! \
xvimagesink


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
 - drop=true для appsink позволяет убрать лаги при последующей передачи из кода куда-либо, возможно потому что отбрасывает лишние кадры. В моем случае на входе в код было 30 кадров, а на выходе из кода с помощью VideoWriter картинка отображалась с 15 кадрами, и вот без drop=true были лаги.  
> **appsink synk=false drop=true**
 - использование плагинов с приставкой **nv** позволяет добиться большей производительности, поскольку вычисления происходят где-то в своей NVMM памяти  
 - Торможение картинки при отображении может быть связано с тем, что у используемого плагина: (я столкнулся при использовании **nvoverlaysink**) должно быть выставлено значение свойства **sync=false**, еще такое свойство есть у **appsink**.  
 - **nvvidconv** копирует из памяти NVMM в память процессора, если слева от него в конвейере **video/x-raw(memory:NVMM)**, а справа просто **video/x-raw**
 >#!/bin/bash
DEC="application/x-rtp, encoding-name=(string)H264, payload=(int)96 ! rtph264depay ! decodebin ! videoconvert ! queue"
gst-launch-1.0 -ve compositor name=comp \
sink_0::width=360 sink_0::height=640 sink_0::xpos=0 \
sink_1::width=360 sink_1::height=640 sink_1::xpos=360 \
sink_2::width=360 sink_2::height=640 sink_2::xpos=720 \
sink_3::width=360 sink_3::height=640 sink_3::xpos=1080 \
! videoconvert ! autovideosink \
udpsrc port=15100 ! $DEC ! comp.sink_0 \
udpsrc port=15121 ! $DEC ! comp.sink_1 \
udpsrc port=15120 ! $DEC ! comp.sink_2 \
udpsrc port=15101 ! $DEC ! comp.sink_3
Внимание! На jetson следует заменить compositor -> nvcompositor 
Это же касается jprgenc -> nvjpeпenc etc...

 - **nvcompositor** позволяет брать изображения сразу с нескольких источников и отправлять тоже куда-то в несколько, делать преобразования размера изображений, или обрезать их и всякое такое


For anyone that comes onto this page, this is my current solution:
MIPI CSI pipeline =
nvarguscamerasrc sensor_id=ID ! video/x-raw(memory:NVMM), width=X, height=Y, format=(string)NV12 ! nvvidconv flip-method=M ! video/x-raw, format=I420, appsink max-buffers=1 drop=true

and UVC pipeline =
v4l2src device=dev ! image/jpeg, width=X, height=Y ! jpegparse ! jpegdec ! nvvidconv flip-method=M ! video/x-raw, format=I420 ! appsink max-buffers=1 drop=true

One optimization is the appsink options: max-buffers=1 and drop=true. These remove an application queuing delay to a good amount.

One other optimization is using format=I420, rather than using BGRx and converting to BGR in vidconv. In my application I would only process the frames after I had grabbed them from all cameras. vidconv is slow because it runs on the CPU. This delay ended up delaying the pipeline. So I use I420 which is supported in nvvidconv and OpenCV. I only use cv::VideoCapture’s grab() function. I use retrieve() later when processing the last frame and I will use cv::cvtColor(image, _image, cv::COLOR_YUV2BGR_I420) so that I can use the frames in BGR as OpenCV needs.

Later on, I am hoping to add nvv4l2decoder, once r32.4.3 is released. @DaneLLL, do you know when r32.4.3 will be released? I noticed r32.4.2b was recently added but I’m guessing that doesn’t have the nvv4l2decoder fix.

In order to get the appropriate image I need, I am using 4 grab() calls to these pipelines. Originally, I was using 7-12 grabs() and it was a mess. This method is much faster and I will update this thread if I find more optimized pipelines for this use case that uses two MIPI CSI and two UVC cameras for instant captures.

