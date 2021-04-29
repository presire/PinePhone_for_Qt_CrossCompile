# PinePhone(Mobian OS)_for_Qt_CrossCompile

# Preface
Here, I use SUSE for my Linux PC and Mobian(Phosh) for my PinePhone.  
When building Qt, please adapt to each user's environment.  
<br>
*If you are using Mobian OS, I think you can do Qt Cross-Compile with similar steps to Raspberry Pi.*  
<br>

# 1. Install the necessary dependencies for PinePhone and SSH Setting (Mobian OS)
Get the latest updates on PinePhone.  

    sudo apt-get update  
    sudo apt-get dist-upgrade  
    sudo shutdown -r now  
<br>

Install SSH server on PinePhone.  

    sudo apt-get install openssh-server  
<br>

Configure the SSH Server to start automatically, and start the SSH Server.  

    sudo systemctl enable ssh  
    sudo systemctl restart ssh  
<br>

Install the dependencies required to build the Qt Library.  

    sudo apt-get install  gdbserver python python3 \  
                          ccache libicu-dev icu-devtools libhd-dev libsctp1 libsctp-dev libzstd1 libzstd-dev \  
                          libinput-bin libinput-dev libts0 libts-bin libts-dev libmtdev1 libmtdev-dev libevdev2 libevdev-dev \  
                          libblkid-dev libffi-dev libglib2.0-dev libglib2.0-dev-bin libmount-dev \  
                          libpcre16-3 libpcre3-dev libpcre32-3 libpcrecpp0v5 libselinux1-dev libsepol1-dev libwacom-dev libassimp-dev libassimp5 \  
                          libfontconfig1-dev libdbus-1-dev libnss3-dev libxkbcommon-dev libjpeg-dev libasound2-dev libudev-dev libxcb-xinerama0 libxcb-xinerama0-dev libpugixml1v5 \  
                          libsqlite3-dev libxslt1-dev libssl-dev \  
                          libatspi2.0-0 libatspi2.0-dev libsctp1 libsctp-dev \  
                          libwayland-bin libwayland-dev libwayland-egl++0 libwayland-egl-backend-dev libwayland-client++0 libwayland-client-extra++0 libwayland-cursor++0 wayland-scanner++ \
                          waylandpp-dev libweston-9-dev libgles2-mesa-dev libegl-dev libgegl-dev libegl1-mesa-dev libgles-dev libwayland-egl1-mesa
<br>

*If you want to use other features, you should also install the following dependencies.*  

* Bluetooth  
bluez bluez-tools libbluetooth-dev	

* Photo  
libjpeg-dev libpng-dev libtiff-dev  

* Codec  
libavcodec-dev libavformat-dev libswscale-dev libv4l-dev libxvidcore-dev libx264-dev  

* Multimedia  
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
libopenal-data libsndio7.0 libopenal1 libopenal-dev  
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
<br>

# 2. Qt Source Code Download (Linux PC)
    wget https://download.qt.io/official_releases/qt/5.15/5.15.2/single/qt-everywhere-src-5.15.2.tar.xz  
    tar xf qt-everywhere-opensource-src-5.15.2.tar.gz  
<br>

# 3. Download the Qt target file from this Github (Linux PC)
    git clone https://github.com/presire/PinePhone_for_Qt_CrossCompile.git
    
    cp -r PinePhone_for_Qt_CrossCompile/linux-pinephone-g++ qt-everywhere-src-5.15.2/qtbase/mkspecs/devices  
<br>

# 4. Setting Linux PC and Download GCC ARM Toolchain (Linux PC)
Install the dependencies required to build the Qt Library for Linux PC. (I use SUSE)  
(Perhaps **texinfo** is unnecessary.)  

    sudo zypper install autoconf automake cmake unzip tar git wget pkg-config gperf gcc gcc-c++ \  
                        gawk bison openssl flex figlet pigz ncurses-devel ncurses5-devel texinfo  
<br>

Download GCC ARM Toolchain. (https://releases.linaro.org/components/toolchain/binaries)  

    wget https://releases.linaro.org/components/toolchain/binaries/7.5-2019.12/aarch64-linux-gnu/gcc-linaro-7.5.0-2019.12-x86_64_aarch64-linux-gnu.tar.xz  
    tar xf gcc-linaro-7.5.0-2019.12-x86_64_aarch64-linux-gnu.tar.xz  
<br>

# 5. Download & Install Wayland-Scanner (Linux PC)
for building **Wayland-Scanner**.  
You need to install **Meson** & **Ninja**.  

    git clone https://github.com/wayland-project/wayland.git  
    cd wayland && mkdir build  

    meson ./build/ --prefix=<Anywhere you like> -Ddocumentation=false  
    ninja -C build/ install  
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

Adjust the symbolic links of downloaded files and directories relative to each other.  
Since fixQualifiedLibraryPaths does not work properly, download and run the script provided.  

    wget https://raw.githubusercontent.com/riscv/riscv-poky/master/scripts/sysroot-relativelinks.py  
    chmod +x sysroot-relativelinks.py  
    
    ./sysroot-relativelinks.py ~/<System Root PinePhone>  
<br>

# 7. Build Qt Source Code (Linux PC)
Build the Qt source code.  

    export PATH="/<Above, Wayland-Scanner's Install Directory>/bin:$PATH"  
    
    PKG_CONFIG_PATH=/<System Root PinePhone>/usr/lib/aarch64-linux-gnu/pkgconfig \  
    PKG_CONFIG_LIBDIR=/<System Root PinePhone>/usr/lib/pkgconfig:/<System Root>/usr/lib/aarch64-linux-gnu/pkgconfig:/<System Root>/usr/share/pkgconfig \  
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
    -sysroot /<System Root PinePhone> \  
    -prefix /<in the future, You develop Qt Software's Directory> \  
    -extprefix /<Qt Library for PinePhone> \  
    -hostprefix /<Qt Tool for Linux PC>  
<br>

If the Configure script succeeds, execute the make or gmake command.  

    make -j $(nproc) or gmake -j $(nproc)  
    
    make install or gmake install  
<br>

# 8. Upload Qt Library (Linux PC)
Deploy the built Qt library to PinePhone.  

    rsync -avz --rsh="ssh" /<Above, Qt Library for PinePhone>/* \  
    <PinePhone's User Name>@<PinePhone's IP Address or Host Name>:/home/<PinePhone's User Name>/InstallSoftware/Qt_5_15_2  
<br>

# 9. Setting Qt LIBRARY Path and etc... (Mobian OS)
Since the Qt Library you uploaded to PinePhone may have root privileges, execute the following command.  

    sudo chown -R <PinePhone's User Name>:<PinePhone's Group Name> ~/InstallSoftware/Qt_5_15_2  
<br>

Add the following content to the bottom line of the .profile.  
  
    export LD_LIBRARY_PATH="/home/<PinePhone's User Name>/InstallSoftware/Qt_5_15_2/lib:$LD_LIBRARY_PATH"  
    export QT_QPA_PLATFORMTHEME="qt5ct"  
    export DISPLAY=":0"  or  export DISPLAY=":0.0"  
    
    export QML_IMPORT_PATH="/home/<PinePhone's User Name>/InstallSoftware/Qt_5_15_2/qml"  
    export QML2_IMPORT_PATH="/home/<PinePhone's User Name>/InstallSoftware/Qt_5_15_2/qml"  
    export QT_PLUGIN_PATH="/home/<PinePhone's User Name>/InstallSoftware/Qt_5_15_2/plugins"  
    export QT_QPA_PLATFORM_PLUGIN_PATH="/home/<PinePhone's User Name>/InstallSoftware/Qt_5_15_2/plugins/platforms"  
    export QML_IMPORT_TRACE=1  
<br>

Update the .profile.  

    source .profile  
    sudo ldconfig  
<br>

Restart PinePhone.  

    sudo shutdown -r now  
<br>

# 10. Use Qt Creator
Launch Qt Creator.  

Qt Creator - [Project] - [Run] - [Run Settings] - [Environment] - [Details] - [Add]  
then, Click [Fetch Device Environment]Button.  
*(when you run the debugger, Be sure to click on [Fetch Device Environment]Button)*  

    Variable - QT_QPA_PLATFORMTHEME  
    Value - qt5ct  
    
    Variable : DISPLAY  
    Value : 0  or   Value : 0.0  
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
<br>

Make sure you can debug Qt project.  
