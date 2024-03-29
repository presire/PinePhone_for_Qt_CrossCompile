# Qt 5 CrossCompile for PinePhone
Revision Date : 2023/11/17<br>
<br><br>

# Preface  
**This article is Qt 5.15 Cross-Compile and Remote Debug for PinePhone.**  
**I am building a cross compile environment with Qt 5.15.2, 5.15.11 (Mobian) and Qt 5.15.2, 5.15.6, 5.15.8, 5.15.11 (Manjaro ARM).**  
<br>
Here, my Linux PC is SUSE Enterprise 15 and openSUSE 15, my PinePhone is Mobian(Phosh) and Manjaro ARM(Phosh).  
When building Qt, please adapt to each user's environment.  
<br>
If you are using Mobian OS, I think that it is similar to the Cross-Compilation procedure for Raspberry Pi.  
<br>
**Note:**  
**If you want to use the latest GCC AArch64 ToolChain,**  
**you need to build GCC yourself by referring to the URL shown below.**  

https://github.com/presire/How_to_create_GCC_ToolChain_for_PinePhone.git  

**It is easy to create and is recommended.**  
<br><br>

# 1. Install the necessary dependencies for PinePhone and SSH Setting (PinePhone)
Create a directory for installing Qt libraries on PinePhone.  
*This directory will be used in the last part of this article.*  

	mkdir -p ~/InstallSoftware/Qt_5_15_2
<br>

Get the latest updates on PinePhone.  

    # Mobian
    sudo apt-get update  
    sudo apt-get dist-upgrade  
    sudo shutdown -r now  

    # Manjaro
    sudo pacman -Syyu
    sudo shutdown -r now
<br>

Install SSH server on PinePhone.  

    # Mobian
    sudo apt-get install openssh-server  

    # Manjaro
    sudo pacman -S --needed openssh
<br>

Configure the SSH Server to start automatically, and start the SSH Server.  

    # Mobian
    sudo systemctl enable ssh  
    sudo systemctl start ssh  

    # Manjaro
    sudo systemctl enable sshd  
    sudo systemctl start sshd  
<br>

Install the dependencies required to build the Qt Library.  

    # Mobian
    sudo apt-get install  unzip ccache pkgconf-bin pkg-config libpkgconf3 pkgconf \  
                          build-essential make autoconf cmake extra-cmake-modules gcc gfortran gdb gdbserver python3 libgtk-3-dev \  
                          libc6 libc6-dev linux-libc-dev glibc-source libc-dev-bin libc-devtools libglib2.0-0 libglib2.0-dev libglib2.0-dev-bin \  
                          libgmp-dev libmpc-dev libmpfr-dev libisl-dev gconf2 gconf2-common libgconf2-dev \  
                          libdbus-1-3 libdbus-1-dev libdbus-c++-bin libdbus-c++-dev libsctp1 libsctp-dev libatspi2.0-0 libatspi2.0-dev \  
                          libzstd1 libzstd-dev libinput-bin libinput-dev libts0 libts-bin libts-dev libmtdev1 libmtdev-dev libevdev2 \  
                          libevdev-dev libicu-dev icu-devtools libblkid1 libblkid-dev libffi8 libffi-dev libmount1 libmount-dev \  
                          libudev1 libudev-dev libhd21 libhd-dev libtsm-dev pcre2-utils libpcre2-32-0 libpcre2-dev libselinux1 libselinux1-dev \  
                          libsepol2 libsepol-dev libwacom9 libwacom-dev libassimp5 libassimp-dev \  
                          libproxy-dev liblttng-ust1 liblttng-ust-common1 liblttng-ust-dev libb2-1 libb2-dev libsdl2-dev \  
                          libfontconfig1 libfontconfig-dev libfontconfig1-dev \  
                          libjpeg-dev libjpeg62-turbo libjpeg62-turbo-dev libjpeg-dev libasound2 libasound2-dev \  
                          sqlite3 libsqlite3-0 libsqlite3-dev libssl3 libssl-dev libnss3 libnss3-dev libxslt1.1 libxslt1-dev \  
                          libpugixml1v5 libpugixml-dev \  
                          libxkbcommon-dev libxcb-xinerama0 libxcb-xinerama0-dev libglut3.12 libglut-dev \  
                          libwayland-bin libwayland-dev libwayland-egl1 libwayland-egl++1 libwayland-egl1-mesa libwayland-egl-backend-dev \  
                          libwayland-client++1 libwayland-client-extra++1 libwayland-cursor++1 wayland-scanner++ wayland-protocols waylandpp-dev \  
                          libosmesa6-dev mesa-common-dev libopengl-dev libgl-dev libglm-dev libgl1-mesa-dev libgle3-dev libgles-dev \  
                          libgles2-mesa-dev libegl-dev libegl1-mesa-dev libgegl-dev libglu1-mesa libglu1-mesa-dev libglw1-mesa-dev \  
                          libglfw3-dev libglew-dev libglx-dev libgbm-dev libdrm-dev libdirectfb-dev \  
                          libglvnd-dev libglvnd-core-dev libglut-dev libopenal-dev libalut-dev \  
                          libweston-10-0 libweston-10-dev                   \  
                          linux-headers-arm64 linux-headers-6.1-sunxi64        # May not be necessary  
    
    # Manjaro
    sudo pacman -S --needed base-devel rsync vi vim util-linux-libs glib2 make cmake unzip pkg-config \
                            gdb gdb-common gdbm gcc gcc-libs gcc-fortran python2 python3 \
                            ccache icu lksctp-tools python-atspi zstd libinput libtsm mtdev \
                            libevdev libffi pcre pcre2 libwacom assimp fontconfig dbus dbus-c++ nss \
                            libxkbcommon alsa-lib libxinerama pugixml sqlite libxslt openssl ffmpeg \
                            wayland wayland-utils wayland-protocols egl-wayland waylandpp \
                            wrapland wlc wayfire glew-wayland glfw-wayland libva1 \
                            mesa mesa-utils glu libglvnd libb2 lttng-ust libproxy
<br>

*If you want to use other features, you should also install the following dependencies.*  

**Mobian**
* Bluetooth  
bluez bluez-tools libbluetooth-dev	

* Photo  
libjpeg-dev libpng-dev libtiff-dev libmng-dev libwebp-dev  

* Codec  
libavcodec-dev libavformat-dev libavutil-dev libavdevice-dev libavfilter-dev libswscale-dev libswresample-dev  
libpostproc-dev libv4l-dev libxvidcore-dev libx264-dev libx265-dev  

* Multimedia  
libwmf-dev  
libgstreamer1.0-0 libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev gstreamer1.0-plugins-base  
gstreamer1.0-plugins-good gstreamer1.0-plugins-ugly gstreamer1.0-plugins-bad  
libgstreamer-plugins-bad1.0-dev gstreamer1.0-libav gstreamer1.0-pulseaudio gstreamer1.0-tools  
gstreamer1.0-alsa gstreamer1.0-x gstreamer1.0-gl gstreamer1.0-gtk3 gstreamer1.0-qt5  
libwayland-dev  

* ALSA audio  
libasound2-dev  

* Pulse audio  
pulseaudio libpulse-dev	 

* OpenAL audio  
libopenal1 libopenal-data libopenal-dev libsndio7.0 libsndio-dev  

* Text to Speech  
flite1-dev libspeechd-dev  

* Qt SerialPort  
libserialport-dev  

* Database  
ODBC       : unixodbc unixodbc-common unixodbc-dev
PostgreSQL : libpq-dev  
MariaDB    : libmariadbclient-dev  
SQLite     : libsqlite3-dev  

* Printer  
libcups2-dev  

* Accessibility  
libatspi2.0-dev  

* SCTP  
libsctp1 libsctp-dev  
(To enable this, add the **-sctp** option when configuring.)  
<br>

**Manjaro**
* Bluetooth  
bluez bluez-tools bluez-libs bluez-utils	

* Photo  
openjpeg2 libjpeg-turbo libpng libtiff libmng  

* Codec  
ffmpeg v4l-utils xvidcore x264 x265  

* Multimedia  
gstreamer gstreamermm gst-plugins-base gst-plugins-base-libs gst-plugins-good
gst-plugins-bad gst-plugins-bad-libs gst-plugins-ugly gst-libav gst-plugin-wpe
gst-plugin-pipewire gst-plugin-gtk qt-gstreamer gst-plugin-qmlgl gst-plugin-opencv  

* ALSA audio  
alsa-lib  

* Pulse audio  
pulseaudio-alsa	 

* OpenAL audio  
openal  

* Text to Speech  
flite  

* Qt SerialPort  
libserialport  

* Database  
PostgreSQL : postgresql-libs libpqxx  
MariaDB    : mariadb-clients mariadb-libs  
SQLite     : sqlite  

* Printer  
libcups  

* Accessibility  
at-spi2-core at-spi2-atk  

* SCTP  
lksctp-tools  
(To enable this, add the **-sctp** option when configuring.)  

* Webengine  
Required : flex bison gperf readline nss libdrm  
Option1 : libxml2 libxml++ libxslt minizip jsoncpp lcms2 libevent protobuf protobuf-c  
Option2(Be careful, as it is unstable.) : opus libvpx  
<br>

Use the rsync command to synchronize the files between Linux PC and PinePhone.  
However, **some of the files to be synchronized require root privileges.**  
<br>
Therefore, add the following settings to the /etc/sudoers file so that all files can be synchronized even by ordinary users.  
With the following settings, the rsync command will be executed with super user privileges if necessary.  

    echo "$USER ALL=NOPASSWD:$(which rsync)" | sudo tee --append /etc/sudoer
<br>

Restart PinePhone just in case.  

    sudo shutdown -r now
    # or
    sudo systemctl reboot
<br>

# 2. Qt Source Code Download and etc... (Linux PC)
    # Qt 5.15.2
    wget https://download.qt.io/official_releases/qt/5.15/5.15.2/single/qt-everywhere-src-5.15.2.tar.xz  
    tar xf qt-everywhere-opensource-src-5.15.2.tar.gz  

    # Qt 5.15.6
    https://download.qt.io/official_releases/qt/5.15/5.15.6/single/qt-everywhere-opensource-src-5.15.6.tar.xz  
    tar xf qt-everywhere-opensource-src-5.15.6.tar.gz  
<br>
Create the directories needed for the build.  

    mkdir -p /<Build directory for Qt Souce Code> \  
             /<System Root PinePhone>             \  
             /<Qt Tool for Linux PC>/bin          \  
             /<Qt Library for PinePhone>          \  
             /<in the future, You develop Qt Software's Directory>  
<br>

# 3. Download the Qt target file from this Github (Linux PC)
    git clone https://github.com/presire/PinePhone_for_Qt_CrossCompile.git
    
    cp -r PinePhone_for_Qt_CrossCompile/linux-pinephone-g++ qt-everywhere-src-5.15.2/qtbase/mkspecs/devices  
<br>

In the downloaded and copied qmake.conf,<br>
comment **DISTRO_OPTS += deb-multi-arch** if you want to use Manjaro ARM.<br>

    vi qt-everywhere-src-5.15.2/qtbase/mkspecs/devices/linux-pinephone-g++/qmake.conf
<br>

    # qt-everywhere-src-5.15.2/qtbase/mkspecs/devices/linux-pinephone-g++/qmake.conf file
    
    ...

    # If you use Manjaro ARM, you need to comment it.
    #DISTRO_OPTS += deb-multi-arch

    ...
<br>

# 4. Setting Linux PC and Download GCC ARM Toolchain (Linux PC)
First, Install the dependencies required to build the Qt Library for Linux PC.  
*(Perhaps **texinfo** is unnecessary.)*  

    sudo zypper install autoconf automake cmake unzip tar git wget pkg-config gperf gcc gcc-c++ \  
                        gawk bison openssl flex figlet pigz ncurses-devel ncurses5-devel texinfo  
<br>

Next, download or install GCC ToolChain for AArch64.  
There are two ways to use the GCC ToolChain for AArch64.  
<br>

## If create GCC ToolChain for PinePhone (Recommend)
Access the URL shown below to create a GCC ToolChain.  
https://github.com/presire/How_to_create_GCC_ToolChain_for_PinePhone  
<br>
**Note :**  
**When building <u>Qt 5.15</u>, the following error occurs when using <u>GCC 11 or later</u>.**  
**However, no modification is required when building Qt 5.15.6.**  

    error: 'numeric_limits' is not a member of 'std'  static_cast<quint16>(numbers.size()) : std::numeric_limits<quint16>::max();

    error: expected primary-expression before '>' token  static_cast<quint16>(numbers.size()) : std::numeric_limits<quint16>::max();

    error: '::max' has not been declared; did you mean 'std::max'?   static_cast<quint16>(numbers.size()) : std::numeric_limits<quint16>::max();
<br>

This is because std::limits has been removed and moved to the limits.h file.  
<br>
If you are building Qt 5.15.2 using GCC 11 or later,  
edit the file "qt-everywhere-src-5.15.2/qtbase/src/corelib/global/qglobal.h" (**Around lines 45-48**) as shown below.  

    vi qt-everywhere-src-5.15.2/qtbase/src/corelib/global/qglobal.h

    // Before Edit
    #ifdef __cplusplus
    #  include <type_traits>
    #  include <cstddef>
    #  include <utility>
    #endif
    
    // After Edit
    #ifdef __cplusplus
    #  include <type_traits>
    #  include <cstddef>
    #  include <utility>
    #include <limits>          // <- Add a line. (include limits.h file)
    #endif
<br>

### If you use Linaro GCC Toolchain
Linaro offers monthly GNU toolchain integration builds that provide users with a snapshot of upstream builds.  
These builds allow developers to immediately test the functionality of pre-built binaries upstream.  
<br>
Download Linaro GCC ARM Toolchain.  
(latest) GCC Tool Chain Link : https://snapshots.linaro.org/gnu-toolchain/  
<br>
GCC Tool Chain 12.3 Link : https://snapshots.linaro.org/gnu-toolchain/12.3-2023.06-1/aarch64-linux-gnu/  
(old) GCC Tool Chain 7.5 Link : https://releases.linaro.org/components/toolchain/binaries/7.5-2019.12/aarch64-linux-gnu/  

    tar xf gcc-linaro-<version>-x86_64_aarch64-linux-gnu.tar.xz  
<br>


# 5. Download & Install Wayland-Scanner (Linux PC)
You need to install **Meson** & **Ninja** for building **Wayland-Scanner**.  

    git clone https://github.com/wayland-project/wayland.git  
    cd wayland && mkdir build  

    meson ./build/ --prefix=<Install Directory for Wayland-Scanner> -Ddocumentation=false  
    ninja -C build/ install  
    
    cp /<Install Directory for Wayland-Scanner>/bin/wayland-scanner /<Qt Tool for Linux PC>/bin  
<br>

# 6. Download PinePhone's System Root (Linux PC)
It is necessary to synchronize with the root directory of PinePhone, create the system root directory.  

    mkdir -p ~/<System Root PinePhone> && \  
             ~/<System Root PinePhone>/usr && \  
             ~/<System Root PinePhone>/usr/share  
<br>

    rsync -avz --rsync-path="sudo rsync" --delete --rsh="ssh" <PinePhone's User Name>@<PinePhone's IP Address or Host Name>:/lib ~/<System Root PinePhone>/  
    rsync -avz --rsync-path="sudo rsync" --delete --rsh="ssh" <PinePhone's User Name>@<PinePhone's IP Address or Host Name>:/usr/lib ~/<System Root PinePhone>/usr  
    rsync -avz --rsync-path="sudo rsync" --delete --rsh="ssh" <PinePhone's User Name>@<PinePhone's IP Address or Host Name>:/usr/include ~/<System Root PinePhone>/usr  
    rsync -avz --rsync-path="sudo rsync" --delete --rsh="ssh" <PinePhone's User Name>@<PinePhone's IP Address or Host Name>:/usr/share/pkgconfig ~/<System Root PinePhone>/usr/share  
<br>

Convert symbolic links of downloaded files and directories from absolute paths to relative paths.

    wget https://raw.githubusercontent.com/riscv/riscv-poky/master/scripts/sysroot-relativelinks.py
    chmod +x sysroot-relativelinks.py
    
    ./sysroot-relativelinks.py <System Root PinePhone>
<br>

# 7. Build Qt Source Code (Linux PC)
**Note**  
When you use **GCC 13**, QtLocation builds may output build errors as shown below.  

    error: 'uint8_t' was not declared in this scope  
<br>

This is because GCC 13 shuffles some internal includes, so ***<code>cstdint</code>*** is not included transiently.  
To use **<code>uint8_t</code>**, ***<code>cstdint</code>*** must be explicitly included.  

    // qt-everywhere-src-5.15.XX/qtlocation/src/3rdparty/mapbox-gl-native/include/mbgl/util/string.hpp  
    // Line 7.  
    #include <cstdint>  // Add  
    
    
    // qt-everywhere-src-5.15.XX/qtlocation/src/3rdparty/mapbox-gl-native/include/mbgl/util/geometry.hpp  
    // Line 6.  
    #include <cstdint>  // Add  
    
    
    // qt-everywhere-src-5.15.XX/qtlocation/src/3rdparty/mapbox-gl-native/src/mbgl/gl/stencil_mode.hpp  
    // Line 4.  
    #include <cstdint>  // Add  
<br>

Build the Qt source code.  

    export PATH="/<Qt Tool for Linux PC>/bin:$PATH"  
    
    PKG_CONFIG_PATH=/<System Root PinePhone>/usr/lib/pkgconfig:/<System Root PinePhone>/usr/lib/aarch64-linux-gnu/pkgconfig \  
    PKG_CONFIG_LIBDIR=/<System Root PinePhone>/usr/lib/pkgconfig:/<System Root>/usr/lib/aarch64-linux-gnu/pkgconfig:/<System Root>/usr/share/pkgconfig \  
    ../<qt-everywhere-src-5.15.2>/configure -v \
    -release \
    -opensource -confirm-license \  
    -opengl es2 \  
    -qpa wayland \  
    -device linux-pinephone-g++ \  
    -device-option CROSS_COMPILE=/<GCC ToolChain Directory>/bin/aarch64-linux-gnu- \  
    -nomake examples -no-compile-examples -nomake tests -make libs -no-use-gold-linker -recheck-all \  
    -skip qtwebengine -skip qtandroidextras -skip qtmacextras -skip qtwinextras \  
    -skip qtscript -skip qtdoc \                                          (If not required)  
    -skip qtgamepad -skip qtpurchasing -skip qtcharts -skip qtsensors \   (If not required)  
    -skip qtlocation -skip qtspeech -skip qtlottie \                      (If not required)  
    -sysroot    /<System Root PinePhone> \  
    -prefix     /<in the future, You develop Qt Software's Directory> \  
    -extprefix  /<Qt Library for PinePhone> \  
    -hostprefix /<Qt Tool for Linux PC>     \
    #-no-gcc-sysroot                             # Comment if you use Linaro GCC Toolchain.
                                                 # If you have built your own GCC Toolchain, you may need to remove comment.
<br>

If the Configure script succeeds, execute the make or gmake command.  

    make -j $(nproc) or gmake -j $(nproc)  
    
    make install or gmake install  
<br>

# 8. Upload Qt Library (Linux PC)
Deploy the built Qt library to PinePhone.  

    rsync -avz --rsh="ssh" --delete /<Above, Qt Library for PinePhone>/* \  
    <PinePhone's User Name>@<PinePhone's IP Address or Host Name>:/home/<PinePhone's User Name>/InstallSoftware/Qt_5_15_2  
<br>

# 9. Setting Qt LIBRARY Path and etc... (PinePhone)
<del>Since the Qt Library you uploaded to PinePhone may have root privileges, execute the following command.</del>  

    <del>sudo chown -R <PinePhone's User Name>:<PinePhone's Group Name> ~/InstallSoftware/Qt_5_15_2</del>  
<br>

Add the following content to the bottom line of the .profile.  

    export QT_QPA_PLATFORMTHEME="qt5ct"  
    export DISPLAY=":0"  or  export DISPLAY=":0.0"  
    
    export QML_IMPORT_PATH="/home/<PinePhone's User Name>/InstallSoftware/Qt_5_15_2/qml"  
    export QML2_IMPORT_PATH="/home/<PinePhone's User Name>/InstallSoftware/Qt_5_15_2/qml"  
    export QT_PLUGIN_PATH="/home/<PinePhone's User Name>/InstallSoftware/Qt_5_15_2/plugins"  
    export QT_QPA_PLATFORM_PLUGIN_PATH="/home/<PinePhone's User Name>/InstallSoftware/Qt_5_15_2/plugins/platforms"  
    export QML_IMPORT_TRACE=1  # (if necessary)  
<br>

**Note:**  
If you write the environment variable LD_LIBRARY_PATH in .profile,  
you need to set again the environment variable LD_LIBRARY_PATH again in Qt Creator,  
[Project] - [Run] - [Environment] when you use Qt Creator for remote debugging.  

    # ~/.profile  
    export LD_LIBRARY_PATH="/home/<PinePhone's User Name>/InstallSoftware/Qt_5_15_2/lib:$LD_LIBRARY_PATH"  
    export LD_LIBRARY_PATH="/home/<PinePhone's User Name>/InstallSoftware/Qt_5_15_2/plugins/qmltooling:$LD_LIBRARY_PATH"  
<br>

I personally recommend is to create a "00-Qt_5_15_2.conf" file in the /etc/ld.so.conf.d directory,  
and write the path as shown below.  

    # /etc/ld.so.conf.d/00-Qt_5_15_2.conf  
    /home/<PinePhone's User Name>/InstallSoftware/Qt_5_15_2/lib  
    /home/<PinePhone's User Name>/InstallSoftware/Qt_5_15_2/plugins/qmltooling  
<br>

Update the .profile or /etc/ld.so.conf.d/00-Qt_5_15_2.conf.  

    # .profile  
    source .profile  
    
    # /etc/ld.so.conf.d/00-Qt_5_15_2.conf  
    sudo ldconfig  
<br>

Restart PinePhone.  

    sudo shutdown -r now  
<br><br>

# 10. Use Qt Creator
## 10.1 Qt Creator General Settings  
Launch Qt Creator.  
<br>

* Setting up the Qt compiler Linaro GCC AArh64 Toolchain.  
Qt Creator - [Tool] - [Option] - [Kits] - [Compiler] -[Add] - [GCC] - [C]  
/<Linaro GCC AArch64 Toolchain's Directory>/bin/aarch64-linux-gnu-gcc  
Qt Creator - [Tool] - [Option] - [Kits] - [Compiler] -[Add] - [GCC] - [C++]  
/<Linaro GCC AArch64 Toolchain's Directory>/bin/aarch64-linux-gnu-g++  

* Setting up the Qt Debugger Linaro GCC AArh64 Toolchain.  
Qt Creator - [Tool] - [Option] - [Kits] - [Debugger] -[Add]  
/<Linaro GCC AArch64 Toolchain's Directory>/bin/aarch64-linux-gnu-gdb  

* Setting up the Qt Qmake for Qt Cross-Compile.  
Qt Creator - [Tool] - [Option] - [Kits] - [Qt version] -[Add]  
/<Qt Tool for Linux PC's Directory>/bin/qmake  

* Setting up the Qt Qmake for Qt Cross-Compile.  
Qt Creator - [Tool] - [Option] - [Kits] - [Kits] -[Add]  
Compiler : The compiler configured above  
Debugger : The debugger configured above  
Qt version : The Qmake configured above  
Qt mkspec : /<Qt Tool for Linux PC's Directory>/mkspecs/devices/linux-pinephone-g++  
Sysroot(It can be set or unset) : /\<System Root PinePhone\>  

* Setting Add GDB Start Command.  
Qt Creator - [Tool] - [Option] - [Debugger] - [GDB]Tab  
Additional Startup Commands : set sysroot target:/  
<br>

## 10.2 Setting up each project in Qt Creator  
Launch Qt Creator and Open or Create Qt Project.  
<br>

Qt Creator - [Project] - [Run] - [Run Settings] - [Environment] - [Details] - [Add]  
then, Click [Fetch Device Environment]Button.  
**<u>when you run the debugger, Be sure to click on [Fetch Device Environment]Button.</u>**  

* Variable - QT_QPA_PLATFORMTHEME  
Value - qt5ct  
    
* Variable : DISPLAY  
Value : 0  or   Value : 0.0  
    
* Variable : LD_LIBRARY_PATH  
<u>(If you write the environment variable LD_LIBRARY_PATH in PinePhone's .profile, you need to set this.)</u>  
Value : /home/<PinePhone's User Name>/InstallSoftware/Qt_5_15_2/lib:/home/<PinePhone's User Name>/InstallSoftware/Qt_5_15_2/plugins/qmltooling  
<br><br>

# 11. Warning and Error related  
## 11.1 Screen display-related errors for Debug  
When debugging, the following error is output.<br>

    Failed to create wl_display (No such file or directory)
    ...
    error: XDG_RUNTIME_DIR not set in the environment
    ...
    qt.qpa.plugin: Could not load the Qt platform plugin "wayland"
<br>

To fix this error,<br>
Qt Creator Side-Menu [Projects] - [Build & Run] - [Run] - [Command line arguments:]entry,<br>
Add the **-platform wayland** option to the [aaa] entry.<br>
<br>

## 11.2 Debug start speed issues  
When you debug remote targets, GDB Debugger is looking in the local PinePhone's System-Root directory for the libraries.  
So just need to tell GDB Debugger to load the remote PinePhone's System-Root from the remote target.  
<br>
<u>but, <code>"set sysroot target:/"</code> takes a long time to start debugging,</u>  
so writing the following setting will speed up the start of debugging.  
<br>
First, Create the directories and symbolic links shown below.  

    mkdir -p /<PinePhone's System-Root Directory>/home/<PinePhone's User Name>/InstallSoftware  
    ln -s /<Qt Library for PinePhone> /<PinePhone's System-Root Directory>/home/<PinePhone's User Name>/InstallSoftware  
    mv /<PinePhone's System-Root Directory>/home/<PinePhone's User Name>/InstallSoftware/<Qt Library for PinePhone> \  
       /<PinePhone's System-Root Directory>/home/<PinePhone's User Name>/InstallSoftware/Qt_5_15_2  
<br>

Next, Write the following settings.  
Qt Creator - [Tool] - [Option] - [Debugger] - [GDB]Tab - [Additional Startup Commands]  

    set sysroot /＜System Root PinePhone＞  
<br>

## 11.3 Floating point warning in GDB  
When you use GCC AArch64 TooChain 7.5 GDB, the following warning is displayed.  
<br>
**The warning shown below**  

	while parsing target description (at line 68): Vector "v8f" references undefined type "ieee_half"  
	Could not load XML target description; ignoring  
<br>

<u>I don't know if this is the right way to do it,</u>  
if you use "GDB for the GCC AArch64 ToolChain 10.2", you won't get the warning output.  
<br>
Download the GCC AArch64 ToolChain 10.2, shown bellow URL.  
You need to select <u>"AArch64 GNU/Linux target (aarch64-none-linux-gnu)"<u>.  
https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-a/downloads  
<br>

**Note :**  
**The best way is to create your own GCC Toolchain for PinePhone.**  
**The URL shown below describes how to create a GCC Toolchain.**  
<br>
**https://github.com/presire/How_to_create_GCC_ToolChain_for_PinePhone.git**  
<br><br>

Make sure you can debug Qt project.  
