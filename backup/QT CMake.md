# QT CMake

文章来源：https://doc.qt.io/qt-6.5/cmake-get-started.html

## 开始使用CMake

CMake是一组工具，用于构建、测试和打包应用程序。与Qt一样，它在所有主要开发平台上都可以使用。它也得到了各种IDE的支持，包括Qt Creator。
在本节中，我们将向您展示在CMake项目中使用Qt的最简单方法。首先，我们将创建一个基本的控制台应用程序。然后，我们将把项目扩展为一个使用Qt小部件的GUI应用程序。
如果您想知道如何构建带有Qt的现有CMake项目，请参阅有关如何使用命令行构建项目的文档。

## 构建一个C++控制台应用程序

CMake项目是由用CMake语言编写的文件定义的。主要的文件叫做`CMakeLists.txt`，通常放在实际程序源代码的同一目录下。
下面是一个使用Qt编写的C++控制台应用程序的典型`CMakeLists.txt`文件：

```cmake
cmake_minimum_required(VERSION 3.16)

project(helloworld VERSION 1.0.0 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

find_package(Qt6 REQUIRED COMPONENTS Core)
qt_standard_project_setup()

qt_add_executable(helloworld
    main.cpp
)

target_link_libraries(helloworld PRIVATE Qt6::Core)
```

让我们逐行来了解一下内容。

```cmake
cmake_minimum_required(VERSION 3.16)
```

`cmake_minimum_required() `指定了应用程序所需的最低CMake版本，Qt本身至少需要CMake版本为3.16。

如果您使用的是静态构建的Qt - 在Qt for iOS和Qt for WebAssembly中这是默认的 - 您需要CMake 3.21.1或更高版本。

```cmake
project(helloworld VERSION 1.0.0 LANGUAGES CXX)
```

project() 设置项目名称和默认项目版本。LANGUAGES 参数告诉 CMake 程序是用 C++ 编写的。

```cmake
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
```

Qt 6 需要一个支持 C++ 版本 17 或更新的编译器。通过设置 CMAKE_CXX_STANDARD 和 CMAKE_CXX_STANDARD_REQUIRED 变量来强制这一要求，如果编译器太旧，CMake 将打印一个错误。

```cmake
find_package(Qt6 REQUIRED COMPONENTS Core)
```

这段话指示CMake查找Qt 6，并导入Core模块。如果CMake无法定位该模块，那么继续下去就没有意义，因此我们设置REQUIRED标志，让CMake在这种情况下中止。

若查找成功，该模块将会设置一些在“[[Module variables](https://doc.qt.io/qt-6.5/cmake-variable-reference.html#module-variables)](https://doc.qt.io/qt-6.5/cmake-variable-reference.html#module-variables)”中有详细说明的CMake变量。此外，它还会导入我们在下文使用的Qt6::Core目标。

为了让find_package操作成功，CMake必须找到Qt安装位置。有多种方式可以告知CMake关于Qt的信息，但最常见且推荐的方法是将CMake缓存变量`CMAKE_PREFIX_PATH`设置为包含Qt 6安装前缀的路径。需要注意的是，Qt Creator会为你自动处理这个问题。（如果用CLION 的话 需要添加这个变量，其他的IDE也是这样）

```cm
qt_standard_project_setup()
```

`qt_standard_project_setup`命令为典型的Qt应用程序设置了项目范围内的默认设置。

这其中的一项操作是将`CMAKE_AUTOMOC`变量设置为`ON`，这意味着CMake会自动配置规则，以便在需要时自动调用Qt的元对象编译器（moc）。有关更多细节，请参阅[[qt_standard_project_setup](https://doc.qt.io/qt-6.5/qt-standard-project-setup.html)](https://doc.qt.io/qt-6.5/qt-standard-project-setup.html)的参考文档。

```cmake
qt_add_executable(helloworld
    main.cpp
)
```

`qt_add_executable()` 函数告诉 CMake 我们想要构建一个名为 helloworld 的可执行文件（而不是库）目标。它是内置的 `add_executable()` 命令的包装，为静态 Qt 构建提供了自动处理链接 Qt 插件、根据平台特定需求定制库名称等附加逻辑。
该目标应从 C++ 源文件 main.cpp 构建。
通常，在这里您不需要列出头文件。这与 qmake 不同，在 qmake 中，需要显式列出头文件，以便它们由元对象编译器（moc）处理。
要创建库，请参见 `qt_add_library()`。

```cmake
target_link_libraries(helloworld PRIVATE Qt6::Core)
```

最后，target_link_libraries命令告诉CMake，通过引用上述find_package()调用导入的Qt6::Core目标，helloworld可执行文件使用了Qt Core。这样做不仅会向链接器添加正确的参数，而且还能确保将正确的包含目录和编译器定义传递给C++编译器。PRIVATE关键字对于可执行目标来说并非严格必需，但明确指定它是良好的实践。如果helloworld是一个库而非可执行文件，则应指定PRIVATE或PUBLIC（如果库在其头文件中引用了来自Qt6::Core的任何内容，则为PUBLIC，否则为PRIVATE）。

## 构建C++ GUI应用程序

在上一节中，我们展示了用于一个简单控制台应用程序的CMakeLists.txt文件。现在我们将创建一个使用Qt Widgets模块的GUI应用程序。

以下是完整项目文件：

```cmake
cmake_minimum_required(VERSION 3.16)

project(helloworld VERSION 1.0.0 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

find_package(Qt6 REQUIRED COMPONENTS Widgets)
qt_standard_project_setup()

qt_add_executable(helloworld
    mainwindow.ui
    mainwindow.cpp
    main.cpp
)

target_link_libraries(helloworld PRIVATE Qt6::Widgets)

set_target_properties(helloworld PROPERTIES
    WIN32_EXECUTABLE ON
    MACOSX_BUNDLE ON
)
```

让我们逐步解析所做的更改。

```cmake
find_package(Qt6 REQUIRED COMPONENTS Widgets)
```

在find_package调用中，我们将Core替换为Widgets。这将定位Qt6Widgets模块，并提供稍后我们将与其链接的Qt6::Widgets目标。

请注意，由于Qt6::Widgets依赖于它，所以应用程序仍会与Qt6::Core进行链接。

qt_standard_project_setup()
除了CMAKE_AUTOMOC之外，qt_standard_project_setup还将CMAKE_AUTOUIC变量设置为ON。这将自动创建规则，在.ui源文件上调用Qt的用户界面编译器(uic)。

```cmake
qt_add_executable(helloworld
    mainwindow.ui
    mainwindow.cpp
    main.cpp
)
```

我们将一个Qt设计师文件(mainwindow.ui)及其相应的C++源文件(mainwindow.cpp)添加到应用程序目标的源文件中。

```cmake
target_link_libraries(helloworld PRIVATE Qt6::Widgets)
```


在target_link_libraries命令中，我们链接到Qt6::Widgets而不是Qt6::Core。

```cmake
set_target_properties(helloworld PROPERTIES
    WIN32_EXECUTABLE ON
    MACOSX_BUNDLE ON
)
```

最后，我们对应用程序目标设置了以下属性：

在Windows上防止创建控制台窗口。
在macOS上创建应用程序包。
有关这些目标属性的更多信息，请参阅CMake官方文档。

## 项目结构化

包含不止一个目标的项目将受益于清晰的项目文件结构。我们将利用CMake的子目录功能来实现这一点。

鉴于我们计划通过增加更多目标来扩展项目，我们将应用程序的源文件移动到一个子目录中，并在那里创建一个新的CMakeLists.txt文件。

项目根目录结构如下：

```
<项目根目录>
├── CMakeLists.txt
└── src
    └── app
        ├── CMakeLists.txt
        ├── main.cpp
        ├── mainwindow.cpp
        ├── mainwindow.h
        └── mainwindow.ui
```

顶层（根目录下的）CMakeLists.txt文件包含了整个项目的设置、find_package调用以及add_subdirectory指令：

```cmake
cmake_minimum_required(VERSION 3.16)

project(helloworld VERSION 1.0.0 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

find_package(Qt6 REQUIRED COMPONENTS Widgets)
qt_standard_project_setup()

add_subdirectory(src/app)
```

在此文件中设置的变量在子目录项目文件中可见。

应用程序的项目文件`src/app/CMakeLists.txt`包含了可执行目标：

```cmake
qt_add_executable(helloworld
    mainwindow.ui
    mainwindow.cpp
    main.cpp
)

target_link_libraries(helloworld PRIVATE Qt6::Widgets)

set_target_properties(helloworld PROPERTIES
    WIN32_EXECUTABLE ON
    MACOSX_BUNDLE ON
)
```

这种结构使得在项目中轻松添加更多目标，例如库或单元测试变得容易。

注意：请将您的项目构建目录添加到您系统上运行的任何防病毒应用的排除目录列表中。

## 编译库文件

随着项目的扩大，你可能希望将应用程序的部分代码转换成一个库，这个库既可以被应用程序本身使用，也可能被单元测试所引用。本节将展示如何创建这样一个库。

目前我们的应用程序直接在main.cpp中包含了业务逻辑代码。我们将按照上一节所述，将这些代码提取到名为"businesslogic"的新静态库中，该库位于子目录"src/businesslogic"中。

为了简化说明，该库仅由一个C++源文件及其对应的头文件组成，这个头文件会被应用程序的main.cpp所包含：

<项目根目录>
├── CMakeLists.txt
└── src
    ├── app
    │   ├── ...
    │   └── main.cpp
    └── businesslogic
        ├── CMakeLists.txt
        ├── businesslogic.cpp
        └── businesslogic.h

现在让我们来看看库的项目文件（src/businesslogic/CMakeLists.txt）的内容。

```cmake
qt_add_library(businesslogic STATIC
    businesslogic.cpp
)
target_link_libraries(businesslogic PRIVATE Qt6::Core)
target_include_directories(businesslogic INTERFACE ${CMAKE_CURRENT_SOURCE_DIR})
```

逐行解释如下：

```cmake
qt_add_library(businesslogic STATIC
    businesslogic.cpp
)
```

`qt_add_library`命令创建了一个名为`businesslogic`的库。稍后，我们将让应用程序链接至这个目标。

`STATIC`关键字表示这是一个静态库。如果我们想要创建共享库或动态库，应当使用`SHARED`关键字。

```cmake
target_link_libraries(businesslogic PRIVATE Qt6::Core)
```

我们有一个静态库，实际上并不需要链接其他库。但由于我们的库使用了QtCore中的类，因此添加了一个对Qt6::Core的链接依赖。这样做会引入必要的QtCore包含路径和预处理器定义。

```cmake
target_include_directories(businesslogic INTERFACE ${CMAKE_CURRENT_SOURCE_DIR})
```

库的API在头文件`businesslogic/businesslogic.h`中定义。通过调用`target_include_directories`，我们可以确保使用我们库的所有目标都会自动添加指向`businesslogic`目录的绝对路径作为包含路径。

这样，在main.cpp中我们就无需使用相对路径来定位`businesslogic.h`，而可以直接编写：

```cpp
#include <businesslogic.h>
```

最后，我们需要在顶级项目文件中添加库的子目录：

```cmake
add_subdirectory(src/app)
add_subdirectory(src/businesslogic)
```

这样就完成了将库添加到主项目的配置。

## 使用库

为了使用上一节中创建的库，我们需要指导CMake链接该库：

```cmake
target_link_libraries(helloworld PRIVATE
    businesslogic
    Qt6::Widgets
)
```

这样可以确保编译main.cpp时能找到`businesslogic.h`头文件。此外，`businesslogic`静态库将成为`helloworld`可执行文件的一部分。

在CMake术语中，库`businesslogic`指定了其使用者（即应用程序）必须满足的使用要求（即包含路径）。`target_link_libraries`命令负责处理这一需求。这意味着当我们在`helloworld`目标中链接`businesslogic`库时，CMake不仅会把库连接到可执行文件中，还会自动处理所需头文件的搜索路径问题，确保正确包含和编译相关代码。同时，我们也链接了`Qt6::Widgets`库，确保应用程序具备Qt Widgets模块的功能。

## 添加资源文件

我们希望在应用程序中显示一些图像，因此使用Qt资源系统来添加它们。

```cmake
qt_add_resources(helloworld imageresources
    PREFIX "/images"
    FILES logo.png splashscreen.png
)
```

`qt_add_resources`命令会自动创建一个包含所引用图像的Qt资源。在C++源代码中，可以通过指定的资源前缀来访问这些图像：

```cpp
logoLabel->setPixmap(QPixmap(":/images/logo.png"));
```

`qt_add_resources`命令的第一个参数可以是变量名或者目标名称。我们推荐使用如上例所示基于目标的变体形式。这意味着当你将目标名`helloworld`作为命令的第一个参数时，CMake会将资源文件集成到目标`helloworld`的构建过程中，并生成相应的`.qrc`资源文件和编译后的资源对象。这样，在你的应用程序中，就可以通过前缀`/images`来访问到`logo.png`和`splashscreen.png`这两张图片资源了。

## 添加翻译

在Qt项目中，字符串的翻译存储在.ts文件中。有关详情，请参阅Qt的国际化指南。

要将.ts文件添加到项目中，请使用`qt_add_translations`命令。

下面的示例向helloworld目标添加了德语和法语两个翻译文件：

```cmake
qt_add_translations(helloworld
    TS_FILES helloworld_de.ts helloworld_fr.ts)
```

这将创建构建系统规则，用于自动从.ts文件生成对应的.qm文件。默认情况下，生成的.qm文件会被嵌入到资源中，并可通过"/i18n"资源前缀访问。

为了更新.ts文件中的条目，可以构建update_translations目标：

```bash
$ cmake --build . --target update_translations
```

若要手动触发.qm文件的生成，可构建release_translations目标：

```bash
$ cmake --build . --target release_translations
```

关于如何影响.ts文件的处理以及将其嵌入资源中的更多细节，请参考`qt_add_translations`的官方文档说明。

`qt_add_translations`命令是一个便捷包装器。如果需要对.ts文件和.qm文件的处理方式有更细致的控制，可以使用底层命令`qt_add_lupdate`和`qt_add_lrelease`。

进一步阅读：

- 官方CMake文档是使用CMake工作的宝贵资源。

- 官方CMake教程涵盖了常见的构建系统任务。

- 《Professional CMake: A Practical Guide》一书为最相关的CMake特性提供了极好的入门介绍。