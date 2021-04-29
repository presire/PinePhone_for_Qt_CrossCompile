# PinePhone(Mobian OS)_for_Qt_CrossCompile

# 1. PinePhone install Dependencies (Mobian OS)
echo "$USER ALL=NOPASSWD:$(which rsync)" | sudo tee --append /etc/sudoers

sudo apt-get update
sudo apt-get dist-upgrade
sudo reboot

sudo apt-get install  gdbserver python python3
ccache libicu-dev icu-devtools libhd-dev libsctp1 libsctp-dev libzstd1 libzstd-dev 
libinput-bin libinput-dev libts0 libts-bin libts-dev libmtdev1 libmtdev-dev libevdev2 libevdev-dev 
libblkid-dev libffi-dev libglib2.0-dev libglib2.0-dev-bin libmount-dev 
libpcre16-3 libpcre3-dev libpcre32-3 libpcrecpp0v5 libselinux1-dev libsepol1-dev libwacom-dev libassimp-dev libassimp5 
libfontconfig1-dev libdbus-1-dev libnss3-dev libxkbcommon-dev libjpeg-dev libasound2-dev libudev-dev libxcb-xinerama0 libxcb-xinerama0-dev libpugixml1v5
libsqlite3-dev libxslt1-dev libssl-dev 
libwayland-bin libwayland-dev libwayland-egl++0 libwayland-egl-backend-dev libwayland-client++0 libwayland-client-extra++0
libwayland-cursor++0 wayland-scanner
waylandpp-dev libweston-9-dev libgles2-mesa-dev libegl-dev libgegl-dev libegl1-mesa-dev libgles-dev libwayland-egl1-mesa

# 2. Qt Source Code Download (Linux PC)
wget https://download.qt.io/official_releases/qt/5.15/5.15.2/single/qt-everywhere-src-5.15.2.tar.xz
tar xf qt-everywhere-opensource-src-5.15.2.tar.gz

# 3. Download the file from this Github (Linux PC)
cp -r linux-pinephone-g++ qt-everywhere-src-5.15.2/qtbase/mkspecs/devices

# 4. Download GCC ARM Toolchain (Linux PC)
Linux PC install Dependencies (I use SUSE)
sudo zypper install autoconf automake cmake unzip tar git wget pkg-config gperf gcc gcc-c++ 
                    gawk bison openssl flex figlet pigz ncurses-devel ncurses5-devel texinfo

Download GCC ARM Toolchain (https://releases.linaro.org/components/toolchain/binaries)
wget https://releases.linaro.org/components/toolchain/binaries/7.5-2019.12/aarch64-linux-gnu/gcc-linaro-7.5.0-2019.12-x86_64_aarch64-linux-gnu.tar.xz
tar xf gcc-linaro-7.5.0-2019.12-x86_64_aarch64-linux-gnu.tar.xz

# 5. Download & Install Wayland-Scanner (Linux PC)
for building Wayland-Scanner, You need Meson & Ninja

git clone https://github.com/wayland-project/wayland.git
cd wayland && mkdir build

meson ./build/ --prefix=<Anywhere you like> -Ddocumentation=false
ninja -C build/ install

# 6. Setting PinePhone's System Root (Linux PC)
rsync -avz --rsync-path="sudo rsync" --delete --rsh="ssh" <PinePhone's User Name>@<PinePhone's IP Address or Host Name>:/lib /<System Root>/
rsync -avz --rsync-path="sudo rsync" --delete --rsh="ssh" <PinePhone's User Name>@<PinePhone's IP Address or Host Name>:/usr/lib /<System Root>/usr
rsync -avz --rsync-path="sudo rsync" --delete --rsh="ssh" <PinePhone's User Name>@<PinePhone's IP Address or Host Name>:/usr/include /<System Root>/usr
rsync -avz --rsync-path="sudo rsync" --delete --rsh="ssh" <PinePhone's User Name>@<PinePhone's IP Address or Host Name>:/usr/share/pkgconfig /<System Root>/usr/share

# 7. Build Qt Source Code (Linux PC)
export PATH="/<Above, Wayland-Scanner's Install Directory>/bin:$PATH"

PKG_CONFIG_PATH=/<System Root>/usr/lib/aarch64-linux-gnu/pkgconfig \
PKG_CONFIG_LIBDIR=/<System Root>/usr/lib/pkgconfig:/<System Root>/usr/lib/aarch64-linux-gnu/pkgconfig:/<System Root>/usr/share/pkgconfig \
../<qt-everywhere-src-5.15.2>/configure -v -release  -opensource -confirm-license \
-opengl es2 \
-qpa wayland \
-device linux-pinephone-g++ \
-device-option CROSS_COMPILE=/<Linaro GCC ARM 7.5 Tool Chain Directory>/bin/aarch64-linux-gnu- \
-nomake examples -no-compile-examples -nomake tests -make libs -no-use-gold-linker -recheck-all \
-skip qtscript -skip qtwebengine -skip qtandroidextras -skip qtmacextras -skip qtwinextras \  (If not required)
-skip qtgamepad -skip qtpurchasing -skip qtcharts -skip qtsensors \   (If not required)
-skip qtlocation -skip qtspeech -skip qtlottie \                      (If not required)
-skip qtdoc \
-sysroot /<System Root> \
-prefix /<in the future, You develop Qt Software's Directory> \
-extprefix /<Qt Library for PinePhone> \
-hostprefix /<Qt Tool for Linux PC>

make -j $(nproc) or gmake -j $(nproc)

make install or gmake install

# 8. Upload Qt Library (Linux PC)
Deploy the built Qt library to PinePhone.

rsync -avz --rsh="ssh" /<Above, Qt Library for PinePhone>/* \\
<PinePhone's User Name>@<PinePhone's IP Address or Host Name>:/home/<PinePhone's User Name>/InstallSoftware/Qt_5_15_2

# 9. Setting Qt LIBRARY Path and etc... (Mobian OS)
.profile
export LD_LIBRARY_PATH="/home/<PinePhone's User Name>/InstallSoftware/Qt_5_15_2/lib:$LD_LIBRARY_PATH"

export QT_QPA_PLATFORMTHEME="qt5ct"
export DISPLAY=":0"  or  export DISPLAY=":0.0"

export QML_IMPORT_PATH="/home/<PinePhone's User Name>/InstallSoftware/Qt_5_15_2/qml"
export QML2_IMPORT_PATH="/home/<PinePhone's User Name>/InstallSoftware/Qt_5_15_2/qml"
export QT_PLUGIN_PATH="/home/<PinePhone's User Name>/InstallSoftware/Qt_5_15_2/plugins"
export QT_QPA_PLATFORM_PLUGIN_PATH="/home/<PinePhone's User Name>/InstallSoftware/Qt_5_15_2/plugins/platforms"
export QML_IMPORT_TRACE=1


source .profile
sudo ldconfig
sudo shutdown -r now

# 10. Use Qt Creator
Qt Creator - [Project] - [Run] - [Run Settings] - [Environment] - [Details] - [Add]
then, Click [Fetch Device Environment]Button.(Be sure to click on it when you run the debugger)

Variable - QT_QPA_PLATFORMTHEME
Value - qt5ct

Variable : DISPLAY
Value : 0  or   Value : 0.0

Make sure you can debug the project.
