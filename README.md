# Qt5.12 LTS with OpenGL for Raspberry
This package installs Qt5.12 LTS "long term support" with desktop OpenGL on a raspberry pi 4 running Raspbian Buster. The package is suitable for compiling desktop-style, windowed Qt apps under X11. The OpenGL support is in software, using Mesa. 

## Install Instructions
To install, download [qt5-opengl-dev_5.12.5_armhf.deb](https://github.com/koendv/qt5-opengl-raspberrypi/releases/tag/v1.0) and type:
```
sudo apt-get update
sudo apt install ./qt5-opengl-dev_5.12.5_armhf.deb
export QT_SELECT=qt5-opengl
```
This installs Qt5 in ```/usr/lib/qt5.12/```, and creates the qtchooser configuration file ```/usr/share/qtchooser/qt5-opengl.conf```

To remove:
```
dpkg -r qt5-opengl-dev
```
This completes the installation instructions.

## Build Notes
These are notes for a native build of Qt5 on a Raspberry Pi 4, 4GB ram. Compilation takes 10 hours.

Beginning with a clean [2019-09-26-raspbian-buster-lite](https://www.raspberrypi.org/downloads/raspbian/)

## Fix Locale
```
perl -pi -e 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/g' /etc/locale.gen
locale-gen en_US.UTF-8
update-locale en_US.UTF-8
```
## Install Prerequisites 
### For mesa:
```
apt-get update
apt-get install libgl1-mesa-dev libglu1-mesa-dev mesa-common-dev
```
### For qt:
```
apt-get install build-essential git libfontconfig1-dev libdbus-1-dev libfreetype6-dev libicu-dev libinput-dev libxkbcommon-dev libsqlite3-dev libssl-dev libpng-dev libjpeg-dev libglib2.0-dev libraspberrypi-dev libcups2-dev libasound2-dev
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
-skip qtwebengine \
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
build:
```
make
```
(wait 10 hours)

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
Depends: qtchooser, libgl1-mesa-dev, libglu1-mesa-dev, mesa-common-dev, libfontconfig1-dev, libdbus-1-dev, libfreetype6-dev, libicu-dev, libinput-dev, libxkbcommon-dev, libsqlite3-dev, libssl-dev, libpng-dev, libjpeg-dev, libglib2.0-dev, libraspberrypi-dev, libcups2-dev, libasound2-dev
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
[Building Qt 5.12 LTS for Raspberry Pi on Raspbian](https://www.tal.org/tutorials/building-qt-512-raspberry-pi)
