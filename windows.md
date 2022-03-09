# win7设置电脑休眠后，鼠标和键盘会唤醒
电脑右键，打开设备管理器，将鼠标、键盘、网卡的电源管理中唤醒电脑取消掉
# windows设置网卡mac
位置在设备管理器》网络适配器》属性 》 高级 》 网络地址

# windows查看软件依赖库
使用everything搜索ldd.exe，发现C:\Program Files\Git\usr\bin\ldd.exe
因此可以使用些软件对目标软件进行ldd，即可列出软件依赖库
# 查看dll版本
右键-属性-详细信息
# ftp
## 电脑访问ftp失败
```shell
windows无法访问此文件夹。请确保输入的文件名是否正确，并且您有权访问此文件
```
- 设置 网络和internet 代理。关闭代理即可
## 打开文件就跳到浏览器
- 设置 搜索internet，打开internet选项。高级，选择启动ftp文件夹视图
# QT
## 在线安装
> https://blog.csdn.net/yanchenyu365/article/details/110949727
- 在官网，注册帐号
- 点击"Download.Try."，选择"Downloads for open source users"
- 安装时：勾选开源协议，并确认，是否商业！

## 编译安装
> https://doc-snapshots.qt.io/qt5-5.15/windows-building.html
- 编译安装会生成qmake.exe,designer.exe等文件，需要将其配置到QT Creator中方可使用。
- 准备环境
  - 安装mingw
  - 安装python
  - 安装perl
  - 安装opengles
  > https://developer.arm.com/-/media/Files/downloads/open-gl-es-emulator/3.0.4/Mali_OpenGL_ES_Emulator-v3.0.4-2-g8d905-Windows-64bit.zip?revision=89a52b4d-c891-4abc-be58-50db8c20bff9?product=OpenGL%20ES%20Emulator,64-bit,,Windows,3.0.4

> http://download.qt.io/archive/qt/
- 下载 qt-everywhere-src-5.15.2.zip
- configure
```shell
configure -prefix %CD%\qtbase -opensource -release -ssl -openssl OPENSSL_INCDIR=-nomake tests -nomake examples
```
- make
```shell
mingw32-make -j4
```
- EGL/egl.h 找不到错误
```shell
In file included from qwindowsintegration.cpp:85:
qwindowseglcontext.h:45:10: fatal error: EGL/egl.h: No such file or directory
 #include <EGL/egl.h>
          ^~~~~~~~~~~
compilation terminated.
mingw32-make[6]: *** [Makefile.Release:3295: .obj/release/qwindowsintegration.o] Error 1
```

  - 解决办法， 使用everything 打开 egl.h，复制文件路径：C:\Users\user\Downloads\qt-everywhere-src-5.15.2\qtbase\include\QtANGLE\EGL
  - 添加到系统变量 CPLUS_INCLUDE_PATH
- python目录有空格错误
```shell
C:\Program Files\Python38\\python.exe C:/Users/user/Downloads/qt-everywhere-src-5.15.2/qtdeclarative/src/3rdparty/masm/yarr/create_regex_tables > .generated\release\RegExpJitTables.h
'C:\Program' 不是内部或外部命令，也不是可运行的程序
或批处理文件。
mingw32-make[4]: *** [Makefile.Release:1059: .generated/release/RegExpJitTables.h] Error 1
mingw32-make[4]: Leaving directory 'C:/Users/user/Downloads/qt-everywhere-src-5.15.2/qtdeclarative/src/qml'
```
  - 解决办法，修改环境变量，将python目录最后一个/去掉
  - 修改C:\Users\user\Downloads\qt-everywhere-src-5.15.2\qtdeclarative\src\qml\Makefile.Release，用""将C:\Program Files\Python38\python.exe括越来。
- undefined reference to `JSC::Yarr::wordUnicodeIgnoreCaseCharCreate()'
.obj/release/YarrInterpreter.o:YarrInterpreter.cpp:(.text+0x6d8): undefined reference to `JSC::Yarr::newlineCreate()'

```shell
.obj/release/YarrInterpreter.o:YarrInterpreter.cpp:(.text+0x654): undefined reference to `JSC::Yarr::wordUnicodeIgnoreCaseCharCreate()'
.obj/release/YarrInterpreter.o:YarrInterpreter.cpp:(.text+0x6d8): undefined reference to `JSC::Yarr::newlineCreate()'
.obj/release/YarrInterpreter.o:YarrInterpreter.cpp:(.text+0x764): undefined reference to `JSC::Yarr::wordcharCreate()'
.obj/release/YarrJIT.o:YarrJIT.cpp:(.text$_ZN3JSC4Yarr11YarrPattern21newlineCharacterClassEv[_ZN3JSC4Yarr11YarrPattern21newlineCharacterClassEv]+0x29): undefined reference to `JSC::Yarr::newlineCreate()'
.obj/release/YarrJIT.o:YarrJIT.cpp:(.text$_ZN3JSC4Yarr11YarrPattern39wordUnicodeIgnoreCaseCharCharacterClassEv[_ZN3JSC4Yarr11YarrPattern39wordUnicodeIgnoreCaseCharCharacterClassEv]+0x29): undefined reference to `JSC::Yarr::wordUnicodeIgnoreCaseCharCreate()'
```
是由于python的头文件没有在系统路径中，解决办法：
1. 添加环境变量 INCLUDE C:\Program Files\Python38\include
2. 删除 C:\Users\user\Downloads\qt-everywhere-src-5.15.2\qtdeclarative\src\qml\.generated\release

## 部署应用
使用windeployqt可以自动将所有依赖的库提取到可执行文件目录
## ssl支持
> qt.network.ssl: QSslSocket: cannot call unresolved function d2i_X509

在https连接时，出现如上错误，原因是编译器没有安装openssl库
解决办法是：
- 如果QT版本1.3及以下，则需要在Qt安装目录找ssleay32.dll和libeay32.dll这两个文件，并拷贝到编译器安装目录
- 如果QT版本1.3以上，则需要在Qt安装目录找libcrypto-1_1-x64.dll和libssl-1_1-x64.dll这两个文件，并拷贝到编译器安装目录
