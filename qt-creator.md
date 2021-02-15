# Build notes for qtcreator 4.14.0
These are notes for a native build of QtCreator 4.14.0 on a Raspberry Pi OS [2021-01-11-raspios-buster-armhf](https://www.raspberrypi.org/software/operating-systems/) (32 bit). 

## Prerequisites

Beginning with a clean Raspberry Pi OS [2021-01-11-raspios-buster-armhf](https://www.raspberrypi.org/software/operating-systems/) (32 bit). 

## Install qt5
```
wget https://github.com/koendv/qt5-opengl-raspberrypi/releases/download/v5.15.2-1/qt5-opengl-dev_5.15.2_armhf.deb
sudo apt-get update
sudo apt --fix-broken  install ./qt5-opengl-dev_5.15.2_armhf.deb
apt-get install build-essential qtchooser
```
## Build qtcreator
```
wget https://download.qt.io/official_releases/qtcreator/4.14/4.14.0/qt-creator-opensource-src-4.14.0.tar.xz
tar xf qt-creator-opensource-src-4.14.0.tar.xz 
mkdir  build-qt-creator
cd build-qt-creator/
export QT_SELECT=qt5.15.2-opengl
qmake ../qt-creator-opensource-src-4.14.0
make -j4
```
Compile time is less than six hours.

## Create debian package
```
mkdir deb
INSTALL_ROOT=$PWD/deb/usr/lib/qt5.12 make install
mkdir deb/DEBIAN
cat > deb/DEBIAN/control <<EOD
Package: qt5-opengl-qtcreator
Version: 4.14.0
Maintainer: Koen <koen@mcvax.org>
Priority: optional
Section: libs
Bugs: https://github.com/koendv/qt5-opengl-raspberrypi/issues
Homepage: https://github.com/koendv/qt5-opengl-raspberrypi
Depends: build-essential, clang
Architecture: armhf
Description: qtcreator IDE for Qt5.15.2 LTS with desktop OpenGL
 qtcreator IDE for Qt5.15.2 LTS, compiled on raspberry pi 4 running Raspberry Pi OS 2021-01-11-raspios-buster-armhf (32 bit).
 Install qt5-opengl-dev_5.15.2_armhf.deb first.
EOD
fakeroot dpkg-deb -b ./deb/ .
```
 This produces the debian package file ```qt5-opengl-qtcreator_4.14.0_armhf.deb```
 
 This completes the build notes.




