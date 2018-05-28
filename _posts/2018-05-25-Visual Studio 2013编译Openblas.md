#### Cmake生成工程

openblas的版本是2.20，cmake-gui生成VS工程，开始编译。编译Windows版本比编译嵌入式平台的Linux-Arm版本还费劲。

![52721440348](https://raw.githubusercontent.com/jdnie/jdnie.github.io/master/public/1527214403481.png)

------

#### 错误0

<br/>error MSB3073: 命令“set local
<br/>perl F:/deepLearn/git/xianyi-OpenBLAS-6d2da63/exports/gensymbol win2k GENERIC dummy 0 0 1 1 0 0 "" "" > F:/deepLearn/git/xianyi-OpenBLAS-6d2da63/build/openblas.def
<br/>if %errorlevel% neq 0 goto :cmEnd
<br/>:cmEnd
<br/>endlocal & call :cmErrorLevel %errorlevel% & goto :cmDone
<br/>:cmErrorLevel
<br/>exit /b %1
<br/>:cmDone
<br/>if %errorlevel% neq 0 goto :VCEnd
<br/>:VCEnd”已退出，代码为 9009。	C:\Program Files (x86)\MSBuild\Microsoft.Cpp\v4.0\V120\Microsoft.CppCommon.targets	127	5	libopenblas

解决方法：
<br/>安装perl

------

#### 错误1
<br/>error MSB3073: 命令“setlocal
<br/>"D:\Program Files (x86)\CMake\bin\cmake.exe" -DBUILD_TYPE=Release -P cmake_install.cmake
<br/>if %errorlevel% neq 0 goto :cmEnd
<br/>:cmEnd
<br/>endlocal & call :cmErrorLevel %errorlevel% & goto :cmDone
<br/>:cmErrorLevel
<br/>exit /b %1
<br/>:cmDone
<br/>if %errorlevel% neq 0 goto :VCEnd
<br/>:VCEnd”已退出，代码为 1。	C:\Program Files (x86)\MSBuild\Microsoft.Cpp\v4.0\V120\Microsoft.CppCommon.targets	132	5	INSTALL
<br/>
<br/>查看输出信息发现是由于VS2013没有权限：
<br/>4>  -- Install configuration: "Release"
<br/>4>  CMake Error at cmake_install.cmake:34 (file):
<br/>4>    file cannot create directory: C:/Program Files (x86)/OpenBLAS/lib.  Maybe
<br/>4>    need administrative privileges.

解决方法：
<br/>以管理员权限运行VS2013

------

#### 错误2
<br/>error MSB6006: “cmd.exe”已退出，代码为 9009。	C:\Program Files (x86)\MSBuild\Microsoft.Cpp\v4.0\V120\Microsoft.CppCommon.targets	170	5	gencblas
<br/>1>  'sed' 不是内部或外部命令，也不是可运行的程序
<br/>1>  或批处理文件。
<br/>3>  'awk' 不是内部或外部命令，也不是可运行的程序
<br/>3>  或批处理文件。

解决方法：
<br/>Windows没有sed和awk命令
<br/>http://sourceforge.net/projects/gnuwin32/files/gawk/3.1.6-1/gawk-3.1.6-1-bin.zip/download
<br/>https://sourceforge.net/projects/gnuwin32/files/sed/4.2.1/
<br/>PATH环境变量中设置sed和awk的安装位置

------

#### 错误3
<br/>1>  sed：-e 表达式 #1，字符 1：未知的命令：“'”
<br/>1>C:\Program Files (x86)\MSBuild\Microsoft.Cpp\v4.0\V120\Microsoft.CppCommon.targets(170,5): error MSB6006: “cmd.exe”已退出，代码为 1。
<br/>3>  awk: 'BEGIN{print
<br/>3>  awk: ^ invalid char ''' in expression
<br/>3>C:\Program Files (x86)\MSBuild\Microsoft.Cpp\v4.0\V120\Microsoft.CppCommon.targets(170,5): error MSB6006: “cmd.exe”已退出，代码为 1。

解决方法：
<br/>CMakeLists.txt中有一行这样的代码：
<br/>COMMAND ${SED} 's/common/openblas_config/g' ${CMAKE_CURRENT_SOURCE_DIR}/cblas.h > "${CMAKE_BINARY_DIR}/cblas.tmp"
<br/>这行代码在Linux的sed命令执行下是没问题的，在Windows的sed命令执行下有问题，实测：
<br/>Linux: sed 's/common/openblas_config/g' cblas.h > cblas.tmp可以正常执行
<br/>Window: sed "s/common/openblas_config/g" cblas.h > cblas.tmp可以正常执行，即单引号换为双引号
<br/>CMakeLists.txt中相应行替换为：
<br/>COMMAND ${SED} "s/common/openblas_config/g" ${CMAKE_CURRENT_SOURCE_DIR}/cblas.h > "${CMAKE_BINARY_DIR}/cblas.tmp"

<br/>awk也有同样的问题：
<br/>CMakeLists.txt中代码是：
<br/>COMMAND ${AWK} 'BEGIN{print \"\#ifndef OPENBLAS_F77BLAS_H\" \; print \"\#define OPENBLAS_F77BLAS_H\" \; print \"\#include \\"openblas_config.h\\" \"}; NF {print}; END{print \"\#endif\"}' ${CMAKE_CURRENT_SOURCE_DIR}/common_interface.h > ${CMAKE_BINARY_DIR}/f77blas.h
<br/>Linux可正常执行的awk命令是：
<br/>awk 'BEGIN{print "#ifndef OPENBLAS_F77BLAS_H" ; print "#define OPENBLAS_F77BLAS_H" ; print "#include \"openblas_config.h\" "}; NF {print}; END{print "#endif"}' common_interface.h > f77blas.h
<br/>Windows可正常执行的awk命令是：
<br/>awk "BEGIN{print \"#ifndef OPENBLAS_F77BLAS_H\" ; print \"#define OPENBLAS_F77BLAS_H\" ; print \"#include \\\"openblas_config.h\\\" \"}; NF {print}; END{print \"#endif\"}" common_interface.h > f77blas.h

<br/>但更为简洁的处理方法是不用sed和awk，直接生成好文件拷贝过去就可以了，也就不用调试复杂的sed和awk命令了，Window版本的sed和awk也不用安装了。

------

#### 错误4
<br/>COMMAND cp "${CMAKE_BINARY_DIR}/cblas.tmp" "${CMAKE_BINARY_DIR}/cblas.h"
<br/>1>  'cp' 不是内部或外部命令，也不是可运行的程序
<br/>1>  或批处理文件。

解决方法：
<br/>Windows的拷贝文件的命令是copy，Linux的拷贝文件的命令是cp。
<br/>然而改成下面就解决问题了吗？No, No, No!
<br/>COMMAND copy "${CMAKE_BINARY_DIR}/cblas.tmp" "${CMAKE_BINARY_DIR}/cblas.h"
<br/>经过调试，正确的写法应该是：
<br/>COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_BINARY_DIR}/cblas.tmp ${CMAKE_BINARY_DIR}/cblas.h