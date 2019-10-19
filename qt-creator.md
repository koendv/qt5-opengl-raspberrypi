# Build notes for qtcreator 4.9.1
These are notes for a native build of QtCreator 4.9.1 on a Raspberry Pi 4, 4GB ram, running Raspbian Buster. 

## Install Instructions
Make sure the companion package ```qt5-opengl-dev_5.12.5_armhf.deb``` is installed on your system:
```
 $ dpkg -l | grep qt5-opengl
ii  qt5-opengl-dev       5.12.5       armhf       Qt5.12 LTS with desktop OpenGL
```
If not, install ```qt5-opengl-dev_5.12.5_armhf.deb``` first.

To install qtcreator 4.9.1, download [qt5-opengl-qtcreator_4.9.1_armhf.deb](https://github.com/koendv/qt5-opengl-raspberrypi/releases/download/v5.12.5-1/qt5-opengl-qtcreator_4.9.1_armhf.deb) type:
```
sudo apt install ./qt5-opengl-qtcreator_4.9.1_armhf.deb
```
This installs qtcreator in ```/usr/lib/qt5.12/bin/```.

To remove:
```
dpkg -r qt5-opengl-qtcreator
```
This completes the installation instructions.

## Prerequisites

Beginning with a clean [2019-09-26-raspbian-buster-lite](https://www.raspberrypi.org/downloads/raspbian/)

```
wget https://github.com/koendv/qt5-opengl-raspberrypi/releases/download/v5.12.5-1/qt5-opengl-dev_5.12.5_armhf.deb
apt-get update
apt install ./qt5-opengl-dev_5.12.5_armhf.deb
apt-get install build-essential qtchooser
```
## Build qtcreator
```
wget http://download.qt.io/official_releases/qtcreator/4.9/4.9.1/qt-creator-opensource-src-4.9.1.tar.xz
tar xf qt-creator-opensource-src-4.9.1.tar.xz 
mkdir  build-qt-creator
cd build-qt-creator/
export QT_SELECT=qt5-opengl
qmake ../qt-creator-opensource-src-4.9.1 -prefix /usr/lib/qt5.12
make
```
Compile time is less than six hours.

## Create debian package
```
mkdir deb
INSTALL_ROOT=$PWD/deb/usr/lib/qt5.12 make install
mkdir deb/DEBIAN
cat > deb/DEBIAN/control <<EOD
Package: qt5-opengl-qtcreator
Version: 4.9.1
Maintainer: Koen <koen@mcvax.org>
Priority: optional
Section: libs
Bugs: https://github.com/koendv/qt5-opengl-raspberrypi/issues
Homepage: https://github.com/koendv/qt5-opengl-raspberrypi
Depends: build-essential
Architecture: armhf
Description: qtcreator IDE for Qt5.12 LTS with desktop OpenGL
 qtcreator IDE for Qt5.12 LTS, compiled for raspberry pi 4 running 2019-09-26-raspbian-buster[-lite.].
 Install qt5-opengl-dev_5.12.5_armhf.deb first.
EOD
fakeroot dpkg-deb -b ./deb/ .
```
 This produces the debian package file ```qt5-opengl-qtcreator_4.9.1_armhf.deb```
 
 This completes the build notes.




