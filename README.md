# Qt5.15.2 LTS with OpenGL for Raspberry
This package installs Qt5.15.2 LTS "long term support" with desktop OpenGL , compiled on a raspberry pi 4 running Raspberry Pi OS [2021-01-11-raspios-buster-armhf](https://www.raspberrypi.org/software/operating-systems/) (32 bit). The package is suitable for compiling desktop-style, windowed Qt apps under X11. The OpenGL support is in software, using Mesa.

## Install Instructions
### Install Qt5.15.2 libraries and includes
To install, download the [qt5-opengl-dev_5.15.2_armhf.deb](https://github.com/koendv/qt5-opengl-raspberrypi/releases) and type:
```
sudo apt update
sudo apt --fix-broken  install ./qt5-opengl-dev_5.15.2_armhf.deb
```
This installs Qt5 in ```/usr/lib/qt5.15.2/```.

This also creates the qtchooser configuration file ```/usr/share/qtchooser/qt5.15.2-opengl.conf```. If *qtchooser* has been installed on the system, you can select Qt5.15.2 LTS with
```
export QT_SELECT=qt5.15.2-opengl
```

To remove:
```
dpkg -r qt5.15.2-opengl-dev
```

### Install Qt-creator 4.14.1
Install Qt5.15.2 first. Then, to install qt-creator in /usr/local/bin, type:

```
wget https://github.com/koendv/qt5-opengl-raspberrypi/releases/download/v5.15.2-1/qt5-opengl-qtcreator_4.14.1_armhf.deb
sudo apt --fix-broken  install ./qt5-opengl-qtcreator_4.14.1_armhf.deb
```

To remove:
```
dpkg -r qt5-opengl-qtcreator
```

This completes the installation instructions.

If you like this, maybe you want to buy me a cup of tea:

[![ko-fi](images/kofibutton.svg)](https://ko-fi.com/Q5Q03LPDQ)

## Build Notes
These are notes for a native build of Qt5 on a Raspberry Pi 4, 8GB ram.

Beginning with a clean Raspberry Pi OS [2021-01-11-raspios-buster-armhf](https://www.raspberrypi.org/software/operating-systems/).

## Prerequisite Packages

Install the following packages:
```
apt update
apt-get install bison build-essential flex git gperf libasound2-dev libcups2-dev libdbus-1-dev libfontconfig1-dev libfreetype6-dev libgl1-mesa-dev libglib2.0-dev libglu1-mesa-dev libharfbuzz-dev libicu-dev libinput-dev libjpeg-dev libnss3-dev libpng-dev libpulse-dev libraspberrypi-dev libsqlite3-dev libssl-dev libx11-dev libx11-xcb-dev libxcb-composite0-dev libxcb-cursor-dev libxcb-damage0-dev libxcb-dpms0-dev libxcb-dri2-0-dev libxcb-dri3-dev libxcb-ewmh-dev libxcb-glx0-dev libxcb-icccm4-dev libxcb-image0-dev libxcb-imdkit-dev libxcb-keysyms1-dev libxcb-present-dev libxcb-randr0-dev libxcb-record0-dev libxcb-render-util0-dev libxcb-render0-dev libxcb-res0-dev libxcb-screensaver0-dev libxcb-shape0-dev libxcb-shm0-dev libxcb-sync-dev libxcb-util0-dev libxcb-xf86dri0-dev libxcb-xfixes0-dev libxcb-xinerama0-dev libxcb-xinput-dev libxcb-xkb-dev libxcb-xrm-dev libxcb-xtest0-dev libxcb-xv0-dev libxcb-xvmc0-dev libxcomposite-dev libxcursor-dev libxdamage-dev libxext-dev libxfixes-dev libxi-dev libxkbcommon-dev libxkbcommon-x11-dev libxrandr-dev libxrender-dev libxtst-dev mesa-common-dev ninja-build
```

## Build qt5
### Download sources
```
mkdir src
cd src
wget https://download.qt.io/official_releases/qt/5.15/5.15.2/single/qt-everywhere-src-5.15.2.tar.xz
tar xf qt-everywhere-src-5.15.2.tar.xz
```
### Get qt5 mkspecs for raspberry
```
git clone https://github.com/oniongarlic/qt-raspberrypi-configuration.git
cd qt-raspberrypi-configuration
make install DESTDIR=../qt-everywhere-src-5.15.2
cd ..
```
### Compile
In src directory, patch qt:
```
patch -p0 << EOD
--- ./qt-everywhere-src-5.15.2/qtscript/src/3rdparty/javascriptcore/JavaScriptCore/wtf/Platform.h.ORIG	2021-02-10 21:12:19.025766762 +0100
+++ ./qt-everywhere-src-5.15.2/qtscript/src/3rdparty/javascriptcore/JavaScriptCore/wtf/Platform.h	2021-02-10 21:13:56.784949313 +0100
@@ -367,7 +367,8 @@
 #    define WTF_CPU_ARM_TRADITIONAL 1
 #    define WTF_CPU_ARM_THUMB2 0
 #  else
-#    error "Not supported ARM architecture"
+#    define WTF_CPU_ARM_TRADITIONAL 1
+#    define WTF_CPU_ARM_THUMB2 0
 #  endif
 #elif CPU(ARM_TRADITIONAL) && CPU(ARM_THUMB2) /* Sanity Check */
 #  error "Cannot use both of WTF_CPU_ARM_TRADITIONAL and WTF_CPU_ARM_THUMB2 platforms"
EOD
```
Configure qt for opengl "desktop".
```
mkdir build-qt
cd build-qt
PKG_CONFIG_LIBDIR=/usr/lib/arm-linux-gnueabihf/pkgconfig:/usr/share/pkgconfig \
../qt-everywhere-src-5.15.2/configure -platform linux-rpi3-g++ \
-v \
-opengl desktop -eglfs \
-no-gtk \
-xcb -xcb-xlib -bundled-xcb-xinput \
-opensource -confirm-license -release \
-reduce-exports \
-force-pkg-config \
-nomake examples -no-compile-examples \
-skip qtwayland \
-no-feature-geoservices_mapboxgl \
-qt-pcre \
-no-pch \
-ssl \
-evdev \
-system-freetype \
-fontconfig \
-glib \
-prefix /usr/lib/qt5.15.2  \
-qpa eglfs
```
Check the [configuration summary](qt_config_summary.txt) says
```
OpenGL:
    Desktop OpenGL ....................... yes
```

To build, in ```build-qt``` type:
```
make -j4
```
Compilation takes less than 18 hours.

### Create debian package
In the  ```src/build-qt``` directory:
```
mkdir deb
INSTALL_ROOT=$PWD/deb make install
```
Create the configuration file for qtchooser:
```
mkdir -p deb/usr/share/qtchooser/
cat > deb/usr/share/qtchooser/qt5.15.2-opengl.conf <<EOD
/usr/lib/qt5.15.2/bin/
/usr/lib/qt5.15.2/lib/
EOD
```
Create the debian control file:
```
mkdir deb/DEBIAN
cat > deb/DEBIAN/control <<EOD
Package: qt5-opengl-dev
Version: 5.15.2
Maintainer: Koen <koen@mcvax.org>
Priority: optional
Section: libs
Bugs: https://github.com/koendv/qt5-opengl-raspberrypi/issues
Homepage: https://github.com/koendv/qt5-opengl-raspberrypi
Depends: bison, build-essential, flex, git, gperf, libasound2-dev, libcups2-dev, libdbus-1-dev, libfontconfig1-dev, libfreetype6-dev, libgl1-mesa-dev, libglib2.0-dev, libglu1-mesa-dev, libharfbuzz-dev, libicu-dev, libinput-dev, libjpeg-dev, libnss3-dev, libpng-dev, libpulse-dev, libraspberrypi-dev, libsqlite3-dev, libssl-dev, libx11-dev, libx11-xcb-dev, libxcb-composite0-dev, libxcb-cursor-dev, libxcb-damage0-dev, libxcb-dpms0-dev, libxcb-dri2-0-dev, libxcb-dri3-dev, libxcb-ewmh-dev, libxcb-glx0-dev, libxcb-icccm4-dev, libxcb-image0-dev, libxcb-imdkit-dev, libxcb-keysyms1-dev, libxcb-present-dev, libxcb-randr0-dev, libxcb-record0-dev, libxcb-render-util0-dev, libxcb-render0-dev, libxcb-res0-dev, libxcb-screensaver0-dev, libxcb-shape0-dev, libxcb-shm0-dev, libxcb-sync-dev, libxcb-util0-dev, libxcb-xf86dri0-dev, libxcb-xfixes0-dev, libxcb-xinerama0-dev, libxcb-xinput-dev, libxcb-xkb-dev, libxcb-xrm-dev, libxcb-xtest0-dev, libxcb-xv0-dev, libxcb-xvmc0-dev, libxcomposite-dev, libxcursor-dev, libxdamage-dev, libxext-dev, libxfixes-dev, libxi-dev, libxkbcommon-dev, libxkbcommon-x11-dev, libxrandr-dev, libxrender-dev, libxtst-dev, mesa-common-dev, ninja-build
Architecture: armhf
Description: Qt5.15.2 LTS with desktop OpenGL
 Qt5.15.2 LTS "long term support" with desktop OpenGL,
 compiled on raspberry pi 4 running Raspberry Pi OS [2021-01-11-raspios-buster-armhf.
 The package is suitable for compiling desktop-style, windowed Qt apps under X11. The OpenGL support is in software, using Mesa.
EOD
```
Create the debian package:
```
fakeroot dpkg-deb -b ./deb/ .
```

 This produces the debian package file ```./qt5-opengl-dev_5.15.2_armhf.deb```

 This completes the build notes.

## See Also

Setting up a [chroot on raspbian](chroot.md).

[Native Build of Qt5 on a Raspberry Pi](https://wiki.qt.io/Native_Build_of_Qt5_on_a_Raspberry_Pi)

[qt-creator](qt-creator.md) notes

[Building Qt 5.12 LTS for Raspberry Pi on Raspbian](https://www.tal.org/tutorials/building-qt-512-raspberry-pi)




