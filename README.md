# onscripter-jh-macos
How to build onscripter-jh in Mac mini and macOS, only for me

## References  
* onscripter-jh_macos_prefix_build_v1.zip  

## How to build onscripter-jh by static link      
```
# bzip2
make
cp *.a ../../jh/lib
cp *.h ../../jh/include 

-------------

# zlib
./configure --prefix=/Users/macmini/work2/jh

# png
./configure --prefix=/Users/macmini/work2/jh --enable-static --disable-shared

# jpeg, not support libtool
mkdir ../../jh/man/man1
./configure
cp *.a ../../jh/lib/
cp *.h ../../jh/include/

# freetype
./configure --prefix=/Users/macmini/work2/jh --enable-static --disable-shared

# ogg
./configure --prefix=/Users/macmini/work2/jh --enable-static --disable-shared

--------------

# vorbis->ogg
# (x) CFLAGS=-I/Users/macmini/work2/jh/include LDFLAGS=-L/Users/macmini/work2/jh  ./configure --disable-shared --enable-static
./configure --prefix=/Users/macmini/work2/jh --enable-static --disable-shared

# SDL2
./configure -prefix=/Users/macmini/work2/jh --disable-shared --enable-static

# SDL2_image
./configure -prefix=/Users/macmini/work2/jh --disable-shared --enable-static

# SDL2_ttf
FT2_CONFIG=/Users/macmini/work2/jh/bin/freetype-config ./configure -prefix=/Users/macmini/work2/jh --disable-shared --enable-static

# SDL2_mixer
./configure -prefix=/Users/macmini/work2/jh --disable-shared --enable-static

# smpeg2
./configure -prefix=/Users/macmini/work2/jh --disable-shared --enable-static

-------------------

# onscripter-jh
PATH=/Users/macmini/work2/jh/bin:$PATH make
# clang: error: unsupported option '-fopenmp'  
```

## Weibo Record  
```
我试了，好像在现在的macos下是没办法编译SDL1的（可能有其他方法），
SDL2则可以，可以打开xcode工程编译生成.a静态库和framework框架。
等以后有机会研究 ​​​

可以在macos上编译运行SDL2程序了，如下所示。方法：
（1）双击dmg后复制framework，SDL1有gcc命令行说明，SDL2没有；
（2）用SDL1的framework编译，虽然可以编译链接动态库，但运行没有效果
（3）用SDL2的framework编译，可以正常运行（复制framework时
需要把头文件复制进SDL2目录中）
（4）由于SDL2没有例子代码（SDL1有，但SDL2的代码用了SDL_test），
可以复制docs/README-winrt.md内的示例，然后参考testdraw2的代码
（5）gcc只需要链接Cocoa framework即可（-framework开关）；
另外不需要额外的SDL_main源文件（对于SDL2而言）

SDL的官方说法是，他们不维护旧版本的macos（应该是指osx 10之前），
全是用最新版的macos来测试——因为他们也搞不到旧版本的macos来测试，
SDL1似乎只能工作于旧版本的macos，所以mac版最好用SDL2——
至于有没有人去改SDL1来适配最新版macos？似乎没有——
别看着我，我不会 ​​​

其实还有一种方法（似乎是因为有人想编译ffmpeg所以要编译SDL1），
可能可以在新版的macos上运行SDL1，因为SDL1有x11的驱动，
那么就不用cocoa了，直接用x11来显示，方法是安装XQuartz，
当然我没有测试过（估计是绕圈的做法）

要在家工作，这几天研究怎么编译macos版的onscripter
（准确来说是onscripter-jh，其实很简单，用linux的套路方法，
写Makefile静态编译，不过要用-framework参数编译链接动态库）。
顺便练一下VHDL和verilog，看能不能把6502的软核代码编译一下——
首先，要看的懂代码 ​​​

现在我可以在macos下用命令行编译c代码去链接SDL2静态库。
之前尝试链接SDL2动态库，因为SDL2动态库可以从dmg复制安装。
但SDL2静态库需要自己configure编译（默认是cocoa后端，
可以去除metal显示后端）。如果链接动态库，需要增加很多个
-framework参数，例如CoreAudio和carbon，
填写这些framework有一个方法是看源码中的xcode工程，
逐个测试（大概需要8个）。最终编译生成的可执行文件
为2M左右

onscripter-jh SDL2在macos的运行效果如图。
我用的方法是configure --prefix方法编译，
需要修改一下代码，大概方法和Linux编译方法类似。
bug也一样，需要把onscripter.cpp里面刷新屏幕的
代码flushDirect改成全屏刷新（原版的局部刷新计算
区域的算法似乎有问题）。如果需要切换全屏，
可以按Win+F键。声音输出也正常。
写Makefile方式编译暂时还没做，以后有机会再做，
下一步是玩一下swift

目前来看，我想自己构建一个ios版的onscripter-jh应该比较麻烦
（当然，也可以直接用onscripter ios的工程），
我慢慢研究，先从SDL2入手。SDL2除了支持命令行编译
（我之前已经编译出macos版的onscripter-jh，
通过configure --prefix方式），还有一种用法是，
它提供了完整的ios工程（在Xcode-iOS目录下），
可以不用做任何工作即可轻松编译，然后我就发现有很多问题：
（1）高版本的SDL2的Xcode/SDLTest（对应macos版）
可能会编译错误，但我用2.0.10则正常
（2）Xcode-iOS/Demos（对应ios版）用模拟器可以运行，
但如果用真机运行，会有两个效果不对，rectangles和touch，
原因不明，可能有bug（我用的是低版本的SDL2）
（3）其余ios例子则可以在真机上运行
（4）用xcode创建bundle id的个数有上限，好像是10个，
否则等7天后（证书也会在6天内过期）
（5）同一台移动设备上安装的调试版app个数有上限，
可能是4个到5个左右（不同的bundle id）
（6）需要修改bundle id中的前缀（com.company），
否则真机调试会报错。不过我记得以前可以在网页上修改
（我已经修改了），但现在好像是不可以在网页上修改
（我以前指定过），有可能是在developer里面
最开始时指定（我猜测）
（7）xcode可以在provisioning profile中选择
xcode managed profile（自动安装证书），
可能是xcode高版本的新功能
```
