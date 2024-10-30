```Bash
sudo apt-get install bison build-essential clang clang-format flex gperf cmake \
libatspi2.0-dev libbluetooth-dev libclang-dev libcups2-dev libdrm-dev \
libegl1-mesa-dev libfontconfig1-dev libfreetype6-dev \
libgstreamer1.0-dev libhunspell-dev libnss3-dev libopengl-dev \
libpulse-dev libssl-dev libts-dev libx11-dev libx11-xcb-dev \
libxcb-glx0-dev libxcb-icccm4-dev libxcb-image0-dev \
libxcb-keysyms1-dev libxcb-randr0-dev libxcb-render-util0-dev \
libxcb-shape0-dev libxcb-shm0-dev libxcb-sync-dev libxcb-util-dev \
libxcb-xfixes0-dev libxcb-xinerama0-dev libxcb-xkb-dev libxcb1-dev \
libxcomposite-dev libxcursor-dev libxdamage-dev libxext-dev \
libxfixes-dev libxi-dev libxkbcommon-dev libxkbcommon-x11-dev \
libxkbfile-dev libxrandr-dev libxrender-dev libxshmfence-dev \
libxshmfence1 llvm ninja-build nodejs python-is-python3 python3
```

  

```Bash
./configure -prefix /usr/local/Qt6
```

  

```Bash
cmake --build . --parallel
```

  

```Bash
sudo apt-get install '^libxcb.*-dev' libx11-xcb-dev libglu1-mesa-dev \
libxrender-dev libxi-dev libxkbcommon-dev libxkbcommon-x11-dev

sudo apt-get install libxcursor-dev libxcomposite-dev libxdamage-dev \
libxrandr-dev libxtst-dev libxss-dev libdbus-1-dev libevent-dev \
libfontconfig1-dev libcap-dev libpulse-dev libudev-dev libpci-dev \
libnss3-dev libasound2-dev libegl1-mesa-dev gperf bison nodejs

sudo apt-get install libasound2-dev libgstreamer1.0-dev \
libgstreamer-plugins-base1.0-dev libgstreamer-plugins-good1.0-dev \
libgstreamer-plugins-bad1.0-dev

sudo apt-get install python3-html5lib
```

  

```Bash
sudo apt purge ibus
```

  

---

### Commands while building Qt 5.15.2 with GCC

1. `mkdir ../qt-5-15-2-build-gcc`
2. `cd ~/qt-5-15-2-build-gcc`
3. `/home/heather/soft/qt/qt-5-15-2/qt-5-15-2-gcc/qt-everywhere-src-5.15.2-fixed-for-gcc/configure -platform linux-g++ -prefix /usr/local/lib/Qt5_15_2_gcc`
4. `time make -j3`
5. `sudo make install`
6. `PATH=/usr/local/lib/Qt5_15_2_gcc/bin:$PATH`
7. `export PATH`
8. `time make -j3 docs`

  

Build time qt all stuff:

![[Untitled 3.png|Untitled 3.png]]

Build time qt docs:

![[Untitled 1 2.png|Untitled 1 2.png]]

---

### Errors while build from src Qt 5.15.2 with GCC (Path in comments - where file appeared)

1. Error while building with GCC:  
    SOLUTION:  
    Added  
    `\#include <limits>`  
      
    [https://stackoverflow.com/questions/68081481/how-can-i-build-qt-5-13-2-with-gcc-11-1-on-windows](https://stackoverflow.com/questions/68081481/how-can-i-build-qt-5-13-2-with-gcc-11-1-on-windows)
    
    ![[added_limits_at_qglobal.png]]
    
2. Errors in qtwebengine while build 5.15.2 with GCC [https://github.com/pytorch/pytorch/issues/70297](https://github.com/pytorch/pytorch/issues/70297) [https://github.com/google/breakpad/commit/605c51ed96ad44b34c457bbca320e74e194c317e](https://github.com/google/breakpad/commit/605c51ed96ad44b34c457bbca320e74e194c317e)  
    SOLUTION:  
    Remove:  
    `static const unsigned kSigStackSize = std::max(16384, SIGSTKSZ);`  
    Added:  
    `const unsigned kSigStackSize = std::max<unsigned>(16384, SIGSTKSZ);`
    
    ![[added_stati_cast_at_exception_handler.png]]
    
3. Errors in qtwebengine while build 5.15.2 with GCC  
    SOLUTION:  
    Added  
    `\#include <limits>`
    
    ![[qt-5-linux-build-fix-error.png]]
    
4. Errors in qtwebengine while build 5.15.2 with GCC  
    SOLUTION:  
    (Path from screenshot + /string_pool.h)  
    Added  
    `\#include <limits>`
    
    ![[added_limits_at_string_pools.png]]
    

  

  

Clang

`sudo apt install libc++abi-dev libstdc++-dev`

/home/heather/soft/qt/qt-5-15-2/qt-5-15-2-clang/qt-everywhere-src-5.15.2-fixed-for-clang/configure -platform linux-clang-libc++ -prefix /usr/local/lib/Qt5_15_2_clang

### Error

`/home/heather/soft/qt/qt-5-15-2/qt-5-15-2-clang/qt-everywhere-src-5.15.2-fixed-for-clang/qtlocation/src/3rdparty/mapbox-gl-native/include/mbgl/util/unique_any.hpp`

![[Untitled 2 2.png|Untitled 2 2.png]]