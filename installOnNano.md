# df -h: Used 4.4 / Avail 8.7
sudo apt update
sudo apt upgrade
sudo apt-get install git cmake python3-dev libhdf5-serial-dev hdf5-tools libatlas-base-dev gfortran
# df -h: Used 5.6 / Avail 7.5

sudo apt-get install python3-pip
pip3 install --upgrade pip
sudo -H pip3 install -U jetson-stats

sudo apt update
sudo apt upgrade
sudo apt-get -y install cuda-toolkit-10-0

#Добавляем пути к cuda и ее библиотекам!!!
export PATH=/usr/local/cuda-10.0/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda-10.0/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}

# Перезагрузим джетсон
sudo apt install build-essential pkg-config unzip yasm checkinstall libjpeg-dev libpng-dev libtiff-dev

sudo apt install libavcodec-dev libavformat-dev libswscale-dev libavresample-dev libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev libxvidcore-dev x264 libx264-dev libfaac-dev libmp3lame-dev libtheora-dev libfaac-dev libmp3lame-dev libvorbis-dev

sudo apt install libopencore-amrnb-dev libopencore-amrwb-dev
sudo apt-get install libdc1394-22 libdc1394-22-dev libxine2-dev libv4l-dev v4l-utils libgtk-3-dev
sudo pip3 install numpy
sudo apt install python3-testresources
sudo apt-get install libtbb-dev

# Optional libraries:
#sudo apt-get install libprotobuf-dev protobuf-compiler
#sudo apt-get install libgoogle-glog-dev libgflags-dev
#sudo apt-get install libgphoto2-dev libeigen3-dev libhdf5-dev doxygen

sudo apt-get install libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev \
libgstreamer-plugins-bad1.0-dev gstreamer1.0-plugins-base gstreamer1.0-plugins-good \
gstreamer1.0-plugins-bad gstreamer1.0-plugins-ugly gstreamer1.0-libav gstreamer1.0-doc \
gstreamer1.0-tools gstreamer1.0-x gstreamer1.0-alsa gstreamer1.0-gl gstreamer1.0-gtk3 \
gstreamer1.0-qt5 gstreamer1.0-pulseaudio
# df -h: Used 8.4 / Avail 4.7

#############################
# Установка jetson-inference (тут внимательнее с сабмодулями)
############################
# Чтобы установить ветку master и она работала без ошибки, что не может извлечь буфер
# Необходимо в файле /home/jetuser1/Documents/jetson-inference/utils/camera/gstCamera.cpp
# примерно в 152 строке, где формируется пайплайн для камеры MIPI, подправить следующее:
(убрать (memory:NVMM) параметр)
#ifdef ENABLE_NVMM
    //ss << "video/x-raw(memory:NVMM) ! ";
    ss << "video/x-raw ! ";
#else
    ss << "video/x-raw ! ";
#endif


#############################
sudo apt-get update
sudo apt-get install git cmake libpython3-dev python3-numpy
cd Documents
git clone https://github.com/dusty-nv/jetson-inference
cd jetson-inference/
git checkout L4T-R32.5.0
git submodule update --init
mkdir build
cd build
cmake ..

# Выбираем нейросетки, которые хотим использовать
# И возможно выбираем PyTorch
# Чтобы не выдал ошибку что не хватает #include "NvInfer.h" или каких-то одноименных, 
# устанавливаем их ниже
sudo apt install libnvinfer-dev libnvparsers-dev libnvonnxparsers-dev libnvinfer-plugin-dev
# Теперь установлен и cudnn в том числе (вместе с libnvinfer-dev)

make -j4

# Необязательно делать install
sudo make install
sudo ldconfig

cd /home/jetuser1/Documents
sudo rm -r jetson-inference/
# df -h: Used 9.7 / Avail 3.4
######

# Перезагрузим джетсон

#############################
# Установка opencv 4.5.2
#############################
cd ~/Documents/
mkdir opencv_build
cd opencv_build
git clone https://github.com/opencv/opencv.git
git clone https://github.com/opencv/opencv_contrib.git
cd opencv_contrib
git checkout 4.5.2
cd ../opencv/
git checkout 4.5.2
mkdir build 
cd build
# df -h: Used 11 / Avail 2.5

# ИЗМЕНИТЬ OPENCV_EXTRA_MODULES_PATH на подходящий
# Возвращаемся к opencv
cmake \
-D CMAKE_BUILD_TYPE=RELEASE \
-D CMAKE_INSTALL_PREFIX=/usr/local \
-D OPENCV_EXTRA_MODULES_PATH=/home/jetuser/Downloads/opencv_build/opencv_contrib/modules \
-D WITH_CUDA=ON \
-D WITH_GSTREAMER=ON \
-D OPENCV_GENERATE_PKGCONFIG=ON \
-D WITH_V4L=ON \
-D OPENCV_ENABLE_NONFREE=ON \
-D ENABLE_FAST_MATH=1  \
-D WITH_CUDNN=ON \
-D OPENCV_DNN_CUDA=ON \
-D CUDA_ARCH_BIN=7.5 \
..

# Если "Configuring done" то круто
# df -h: Used 11 / Avail 2.3

make -j4
sudo make install
sudo /bin/bash -c 'echo "/usr/local/lib" >> /etc/ld.so.conf.d/opencv.conf'
sudo ldconfig
# df -h: Used 12 / Avail 1.3
# Перезагрузим джетсон

#Удаляем папку opencv_build с исходниками, поскольку opencv уже установили
# df -h: Used 9.9 / Avail 3.2

#########################################
# Установка PyTorch для Pose estimation #
#########################################
# скачиваем нужную версию исходя из версии jetpack отсюда https://forums.developer.nvidia.com/t/pytorch-for-jetson-version-1-10-now-available/72048
# затем
sudo apt-get install python3-pip libopenblas-base libopenmpi-dev 
pip3 install Cython
pip3 install numpy ./torch-1.4.0-cp36-cp36m-linux_aarch64.whl
sudo apt-get install libjpeg-dev zlib1g-dev libpython3-dev libavcodec-dev libavformat-dev libswscale-dev

# Версия torch vision исходя из версии pyTorch
git clone --branch v0.5.0 https://github.com/pytorch/vision torchvision
cd torchvision/
export BUILD_VERSION=0.5.0
python3 setup.py install --user


#Еще надо установить deepstream














