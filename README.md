# Qt5.12 LTS with OpenGL for Raspberry
This package installs Qt5.12 LTS "long term support" with desktop OpenGL on a raspberry pi 4 running Raspbian Buster. The package is suitable for compiling desktop-style, windowed Qt apps under X11. The OpenGL support is in software, using Mesa. 

Optionally the QtCreator 4.9.1 IDE can be installed as well.

## Install Instructions
### Install Qt5.12 libraries and includes
To install, download [qt5-opengl-dev_5.12.5_armhf.deb](https://github.com/koendv/qt5-opengl-raspberrypi/releases/download/v5.12.5-1/qt5-opengl-dev_5.12.5_armhf.deb) and type:
```
sudo apt-get update
sudo apt install ./qt5-opengl-dev_5.12.5_armhf.deb
```
This installs Qt5 in ```/usr/lib/qt5.12/```.

This also creates the qtchooser configuration file ```/usr/share/qtchooser/qt5-opengl.conf```. If *qtchooser* has been installed on the system, you can select Qt5.12 LTS with
```
export QT_SELECT=qt5-opengl
```

To remove:
```
dpkg -r qt5-opengl-dev
```
### Install qtcreator
If, after installing Qt5.12 libraries and includes, you wish to install the qtcreator IDE as well, download [qt5-opengl-qtcreator_4.9.1_armhf.deb](https://github.com/koendv/qt5-opengl-raspberrypi/releases/download/v5.12.5-1/qt5-opengl-qtcreator_4.9.1_armhf.deb) and type:
```
sudo apt install ./qt5-opengl-qtcreator_4.9.1_armhf.deb
```
This installs qtcreator in ```/usr/lib/qt5.12/bin/```.

To remove:
```
dpkg -r qt5-opengl-qtcreator
```

This completes the installation instructions.

## Build Notes
These are notes for a native build of Qt5 on a Raspberry Pi 4, 4GB ram. 

Beginning with a clean [2019-09-26-raspbian-buster-lite](https://www.raspberrypi.org/downloads/raspbian/)

## Prerequisite Packages

For Mesa:
```
apt-get update
apt-get install libgl1-mesa-dev libglu1-mesa-dev mesa-common-dev
```

For Qt:
```
apt-get install build-essential git libfontconfig1-dev libdbus-1-dev libfreetype6-dev libicu-dev libinput-dev libxkbcommon-dev libsqlite3-dev libssl-dev libpng-dev libjpeg-dev libglib2.0-dev libraspberrypi-dev libcups2-dev libasound2-dev
```

For QtWebengine:
```
apt-get install libfontconfig1-dev libfreetype6-dev libx11-dev libxext-dev libxfixes-dev libxi-dev libxrender-dev libxcb1-dev libx11-xcb-dev libxcb-glx0-dev libxkbcommon-x11-dev libxcb-keysyms1-dev libxcb-image0-dev libxcb-shm0-dev libxcb-icccm4-dev libxcb-sync0-dev libxcb-xfixes0-dev libxcb-shape0-dev libxcb-randr0-dev libxcb-render-util0-dev libnss3-dev libxcomposite-dev libxcursor-dev libxtst-dev libxrandr-dev gperf bison flex ninja-build 
```

## Build qt5
### Download sources
```
mkdir src
cd src
wget http://download.qt.io/official_releases/qt/5.12/5.12.5/single/qt-everywhere-src-5.12.5.tar.xz
tar xf qt-everywhere-src-5.12.5.tar.xz
```
### Get qt5 mkspecs for raspberry
```
git clone https://github.com/oniongarlic/qt-raspberrypi-configuration.git
cd qt-raspberrypi-configuration
make install DESTDIR=../qt-everywhere-src-5.12.5
cd ..
```
### Compile
In src directory, patch qt:
```
ls qt-everywhere-src-5.12.5
patch -p0 << EOD
diff -rBNu qt-everywhere-src-5.12.5/qtbase/src/plugins/platforms/xcb/gl_integrations/xcb_egl/qxcbeglwindow.cpp qt-everywhere-src-5.12.5.OLD/qtbase/src/plugins/platforms/xcb/gl_integrations/xcb_egl/qxcbeglwindow.cpp
--- qt-everywhere-src-5.12.5/qtbase/src/plugins/platforms/xcb/gl_integrations/xcb_egl/qxcbeglwindow.cpp	2019-09-03 20:52:35.000000000 +0200
+++ qt-everywhere-src-5.12.5.OLD/qtbase/src/plugins/platforms/xcb/gl_integrations/xcb_egl/qxcbeglwindow.cpp	2019-09-30 16:03:17.831619246 +0200
@@ -93,7 +93,7 @@
 {
     QXcbWindow::create();
 
-    m_surface = eglCreateWindowSurface(m_glIntegration->eglDisplay(), m_config, m_window, 0);
+    m_surface = eglCreateWindowSurface(m_glIntegration->eglDisplay(), m_config, (void*)m_window, 0);
 }
 
 QT_END_NAMESPACE
diff -rBNu qt-everywhere-src-5.12.5/qtscript/src/3rdparty/javascriptcore/JavaScriptCore/wtf/Platform.h qt-everywhere-src-5.12.5.OLD/qtscript/src/3rdparty/javascriptcore/JavaScriptCore/wtf/Platform.h
--- qt-everywhere-src-5.12.5/qtscript/src/3rdparty/javascriptcore/JavaScriptCore/wtf/Platform.h	2019-08-23 12:28:19.000000000 +0200
+++ qt-everywhere-src-5.12.5.OLD/qtscript/src/3rdparty/javascriptcore/JavaScriptCore/wtf/Platform.h	2019-10-01 15:52:01.966516814 +0200
@@ -367,7 +368,8 @@
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
../qt-everywhere-src-5.12.5/configure -platform linux-rpi3-g++ \
-v \
-opengl desktop -eglfs \
-no-gtk \
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
-prefix /usr/lib/qt5.12  \
-qpa eglfs
```
Check the [configuration summary](summary.txt)

To reduce memory usage when compiling, edit ```build-qt/qtwebengine/src/core/Makefile.gn_run``` and change the options to *ninja* from 
```
build-qt/qtwebengine/src/3rdparty/ninja/ninja -v -C 
```
to
```
build-qt/qtwebengine/src/3rdparty/ninja/ninja -j2 -v -C 
```

To build, in ```build-qt``` type:
```
make
```
Compilation takes less than a day.

### Create debian package
In the  ```src/build-qt``` directory:
```
mkdir deb
INSTALL_ROOT=$PWD/deb make install
```
Create the configuration file for qtchooser:
```
mkdir -p deb/usr/share/qtchooser/
cat > deb/usr/share/qtchooser/qt5-opengl.conf <<EOD
/usr/lib/qt5.12/bin/
/usr/lib/qt5.12/lib/
EOD
```
Create the debian control file:
```
mkdir deb/DEBIAN
cat > deb/DEBIAN/control <<EOD
Package: qt5-opengl-dev
Version: 5.12.5
Maintainer: Koen <koen@mcvax.org>
Priority: optional
Section: libs
Bugs: https://github.com/koendv/qt5-opengl-raspberrypi/issues
Homepage: https://github.com/koendv/qt5-opengl-raspberrypi
Depends: libgl1-mesa-dev, libglu1-mesa-dev, mesa-common-dev, libfontconfig1-dev, libdbus-1-dev, libfreetype6-dev, libicu-dev, libinput-dev, libxkbcommon-dev, libsqlite3-dev, libssl-dev, libpng-dev, libjpeg-dev, libglib2.0-dev, libraspberrypi-dev, libcups2-dev, libasound2-dev, libfontconfig1-dev, libfreetype6-dev, libx11-dev, libxext-dev, libxfixes-dev, libxi-dev, libxrender-dev, libxcb1-dev, libx11-xcb-dev, libxcb-glx0-dev, libxkbcommon-x11-dev, libxcb-keysyms1-dev, libxcb-image0-dev, libxcb-shm0-dev, libxcb-icccm4-dev, libxcb-sync0-dev, libxcb-xfixes0-dev, libxcb-shape0-dev, libxcb-randr0-dev, libxcb-render-util0-dev, libnss3-dev, libxcomposite-dev, libxcursor-dev, libxtst-dev, libxrandr-dev, gperf, bison, flex, ninja-build
Architecture: armhf
Description: Qt5.12 LTS with desktop OpenGL
 Qt5.12 LTS "long term support" with desktop OpenGL,
 compiled for raspberry pi 4 running 2019-09-26-raspbian-buster[-lite.]
 The package is suitable for compiling desktop-style, windowed Qt apps under X11. The OpenGL support is in software, using Mesa.
EOD
```
Create the debian package:
```
fakeroot dpkg-deb -b ./deb/ .
```
 
 This produces the debian package file ```qt5-opengl-dev_5.12.5_armhf.deb```
 
 This completes the build notes.
 
## See Also

For convenience, I build software like this in a *chroot* and bundle the applications as an [AppImage](http://www.appimage.org). Setting up a [chroot on raspbian](chroot.md). 

[Building Qt 5.12 LTS for Raspberry Pi on Raspbian](https://www.tal.org/tutorials/building-qt-512-raspberry-pi)




