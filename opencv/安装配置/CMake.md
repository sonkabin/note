# CMake

1. error。参考[how-to-set-the-cmake-prefix-path](https://stackoverflow.com/questions/8019505/how-to-set-the-cmake-prefix-path)

    ```shell
    cmake .
    # 报错信息'''
    CMake Error at CMakeLists.txt:3 (find_package):
      By not providing "FindOpenCV.cmake" in CMAKE_MODULE_PATH this project has
      asked CMake to find a package configuration file provided by "OpenCV", but
      CMake did not find one.
    
      Could not find a package configuration file provided by "OpenCV" with any
      of the following names:
    
        OpenCVConfig.cmake
        opencv-config.cmake
    
      Add the installation prefix of "OpenCV" to CMAKE_PREFIX_PATH or set
      "OpenCV_DIR" to a directory containing one of the above files.  If "OpenCV"
      provides a separate development package or SDK, be sure it has been
      installed.
    
    -- Configuring incomplete, errors occurred!
    See also "/home/sonkabin/workspace/opencv/test/CMakeFiles/CMakeOutput.log".
    '''
    
    # 解决方案
    cmake  -DCMAKE_PREFIX_PATH=~/workspace/opencv/opencv/build .
    ```

2. 配置

    ```cmake
    cmake_minimum_required(VERSION 2.8) # CMake的最低版本
    project( DisplayImage ) # 定义项目名称，保存在 POJECT_NAME 变量中
    find_package( OpenCV REQUIRED ) # 查找依赖项、版本、必须还是可选
    include_directories( ${OpenCV_INCLUDE_DIRS} ) # 向环境中添加指定库的头文件
    add_executable( ${PROJECT_NAME} DisplayImage.cpp ) # 从DisplayImage.cpp文件创建一个可执行命令，将其命名为与项目的名称相同
    target_link_libraries( DisplayImage ${OpenCV_LIBS} ) # 将可执行文件链接到所需库的函数中，这里是需要opencv库
    
    # 补充
    
    # 定义库的源文件及其名称。生成一个共享（.so[mac/linux] or .dll[win）或静态库(.a[mac/linux] or .lib[win])
    add_library(Hello hello.cpp hello.h) 
    
    # 在终端显示一条消息
    message("OpenCV version: ${OpenCV_VERSION}")
    
    # 创建变量
    set(SRC main.cpp)
    ```

3. 生成可执行文件

   ```shell
   cd <DisplayImage_directory>
   cmake  -DCMAKE_PREFIX_PATH=~/workspace/opencv/opencv/build .
   make
   ```

4. 一个包含子目录的脚本

    ```shell
    CMakeList.txt
    main.cpp
    utils/
    	CMakeList.txt
    	computeTime.cpp
    	computeTime.h
    	logger.cpp
    	logger.h
    	plotting.cpp
    	plotting.h
    ```

    ```cmake
    # utils文件夹的CMakeList.txt
    set(UTILS_LIB_SRC computeTime.cpp logger.cpp plotting.cpp)
    add_library(Utils ${UTILS_LIB_SRC})
    target_link_libraries(Utils PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}) # 允许主项目检测所有头文件
    ```

    ```cmake
    # 根文件夹下的CMakeList.txt
    # ...省略部分
    
    add_subdirectory(utils) # 让Cmake分析子文件夹下的 CMakeList.txt
    
    # option函数创建新的变量，可通过cmake命令行更改此变量。它使用户能够决定编译时功能
    option(WITH_LOG "Build with output logs and images in tmp" OFF)
    if(WITH_LOG)
    	add_definitions(-DLOG) # 定义LOG宏
    endif(WITH_LOG)
    
    target_link_libraries(${PROJECT_NAME} ${OpenCV_LIBS} Utils)
    ```

5. 

