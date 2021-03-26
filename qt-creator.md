# Build notes for qtcreator 4.14.1
These are notes for a native build of QtCreator 4.14.1 on a Raspberry Pi OS [2021-01-11-raspios-buster-armhf](https://www.raspberrypi.org/software/operating-systems/) (32 bit).

## Prerequisites

Beginning with a clean Raspberry Pi OS [2021-01-11-raspios-buster-armhf](https://www.raspberrypi.org/software/operating-systems/) (32 bit).

## Install qt5 with desktop OpenGL
```
wget https://github.com/koendv/qt5-opengl-raspberrypi/releases/download/v5.15.2-1/qt5-opengl-dev_5.15.2_armhf.deb
sudo apt-get update
sudo apt --fix-broken  install ./qt5-opengl-dev_5.15.2_armhf.deb
sudo apt-get install build-essential qtchooser cmake ninja-build elfutils libelf-dev libdw-dev libasm-dev libzstd-dev
```
## Build qtcreator
```
wget https://download.qt.io/official_releases/qtcreator/4.14/4.14.1/qt-creator-opensource-src-4.14.1.tar.xz
tar xf qt-creator-opensource-src-4.14.1.tar.xz

patch -p0 <<EOD
--- qt-creator-opensource-src-4.14.1.ORIG/CMakeLists.txt	2021-02-23 09:29:55.000000000 +0000
+++ qt-creator-opensource-src-4.14.1/CMakeLists.txt	2021-03-19 13:04:39.696303218 +0000
@@ -1,5 +1,7 @@
 cmake_minimum_required(VERSION 3.10)
 
+set(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)
+
 ## Add paths to check for cmake modules:
 list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")
EOD

mkdir  build-qt-creator
cd build-qt-creator/
export QT_SELECT=qt5.15.2-opengl
export Qt5_DIR=/usr/lib/qt5.15.2/
cmake -DPYTHON_EXECUTABLE=/usr/bin/python3 ../qt-creator-opensource-src-4.14.1 | tee qtcreator-cmake.txt
make -j4
```

Check [Configuration output](qtcreator_config_summary.txt)

70 minutes compiling...

## Create debian package
```
mkdir deb
DESTDIR=$PWD/deb make install
mkdir deb/DEBIAN
cat > deb/DEBIAN/control <<EOD
Package: qt5-opengl-qtcreator
Version: 4.14.1
Maintainer: Koen <koen@mcvax.org>
Priority: optional
Section: libs
Bugs: https://github.com/koendv/qt5-opengl-raspberrypi/issues
Homepage: https://github.com/koendv/qt5-opengl-raspberrypi
Depends: qt5-opengl-dev, build-essential, qtchooser, cmake, ninja-build, elfutils, libelf-dev, libdw-dev, libasm-dev, libzstd-dev
Architecture: armhf
Description: qtcreator IDE for Qt5.15.2 LTS with desktop OpenGL
 qtcreator IDE for Qt5.15.2 LTS, compiled on raspberry pi 4 running Raspberry Pi OS 2021-01-11-raspios-buster-armhf (32 bit).
 Install qt5-opengl-dev_5.15.2_armhf.deb first.
EOD
fakeroot dpkg-deb -b ./deb/ .
```
This produces the debian package file ```qt5-opengl-qtcreator_4.14.1_armhf.deb```

This completes the build notes.
