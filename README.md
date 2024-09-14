# Windows installation opencv cuda cpp instructions

instal CMake, during installation choose add to path or after add to system environmental variable, cmake:  C:\Program Files\CMake\bin

Trying to install cuda 11.8 with cudnn 9.1.1 installer (in the installer only 11.8) and adding to path the bin dir:
C:\Program Files\NVIDIA\CUDNN\v9.1\bin\11.8
C:\Program Files\NVIDIA\CUDNN\v9.1\include\11.8
C:\Program Files\NVIDIA\CUDNN\v9.1\lib\11.8\x64

open cmake gui
Browse source:"C:\Users\hadar\Documents\opencvBuilds\opencv-4.5.5"
browse build: C:\Users\hadar\Documents\opencvBuilds\build
press configure: vs2019, in the second line choose x64, then press finish

once finished, choose from search:
1. with_cuda
2. opencv_dnn_cuda
3. enable_fast_math
4. opencv_extra_modules_path
	opencv_conrib/ modules
5. build_opencv_world

press configure

choose from search

6. cuda_fast_math
   
8. cuda_arch_bin
	6.1 (for GTX 1050)
   
10. cmake_configuration_types
	Release

press configure

press Generate

cmd: 
cmake --build "C:\Users\hadar\Documents\opencvBuilds\build"  --target INSTALL --config Release

should see that CMakeCache.txt was creted in C:\Users\hadar\opencv\build so the binaries should be created there 

add to path:
C:\Users\hadar\Documents\opencvBuilds\build\install\x64\vc16\bin
C:\Users\hadar\Documents\opencvBuilds\build\install\x64\vc16\lib

open vs code, install extensions: c/c++ and cmake tools, might need to restart
first time, press ctrl+shift+p for command pallete and search cmake:configure, choose vs 2019 86 x64.
then, ctrl+shift+p search cmake: quick start -> project name: testgpucpp -> c++ -> executable.

## python cv2 (vscode on windows)
os.add_dll_directory(r"C:\Program Files\NVIDIA\CUDNN\v9.1\bin\11.8")  # For cuDNN DLLs

os.add_dll_directory(r"C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v11.8\bin")  # For cuda DLLs

os.add_dll_directory(r"C:\opencv\install\x64\vc16\bin")  # For OpenCV DLLs

"ImportError: dynamic module does not define module export function (PyInit_vmd_lib)"

"ImportError: DLL load failed while importing cv2: The specified module could not be found."

## In winodws, using visual studio, there were issues using cmakelists.txt with vs code, only visual studio 19 works reliably.

# working on visual studio 2019:

right click on the project -> properties:
1. On top set Configuration: Active(Release), Platform: Active(x64)
2. Configuration properties -> C/C++:
include Directories -> scroll down -> edit -> create new line -> 
C:\Users\hadar\Documents\opencvBuilds\build\install\include ; C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v11.8\include\

press Apply

3. Configuration properties -> Linker:
General -> Additional Include Directories -> 
C:\Users\hadar\Documents\opencvBuilds\build\install\x64\vc16\lib ; C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v11.8\lib\x64

Input -> Additional dependencies -> scroll down -> edit -> opencv_world455.lib (or whatever opencv version opencv_worldxxx.lib you have in C:\Users\hadar\Documents\opencvBuilds\build\install\x64\vc16\lib)

press Apply

For cuda I also added every npp.lib after I had linker error which had npp in it:
cuda.lib
cudart.lib
nppc.lib
nppial.lib
nppicc.lib
nppidei.lib
nppif.lib
nppig.lib
nppim.lib
nppist.lib
nppisu.lib
nppitc.lib
npps.lib

press Apply


# example CMakeLists.txt

that works after many tries on the cmake versions: 3.0 3.5 3.26, but have to be below 3.27 because of deprecated package

cmake_minimum_required(VERSION 3.0)
project(testgpuopencv)

## Find OpenCV package
find_package(OpenCV REQUIRED)
include_directories(${OpenCV_INCLUDE_DIRS})

## Add executable
add_executable(${PROJECT_NAME} main.cpp)

target_link_libraries(${PROJECT_NAME} ${OpenCV_LIBS})

## Display messages for debugging
message("--------------- OpenCV_INCLUDE_DIRS: " ${OpenCV_INCLUDE_DIRS})

message("--------------- OpenCV_LIBS: " ${OpenCV_LIBS})

message("--------------- PROJECT_NAME: " ${PROJECT_NAME})

message("--------------- CMAKE_BUILD_TYPE: " ${CMAKE_BUILD_TYPE})


# another CMakeLists.txt example
  
'''make sure the VERSION < 3.27 (may have to play with the differnet versions), any higher will lead to deprecation issues such as
 Could not find a package configuration file provided by "CUDA" (requested
 version 11.8) with any of the following names:
	CUDAConfig.cmake
	cuda-config.cmake
'''
cmake_minimum_required(VERSION 3.5)
project(testgpucpp VERSION 0.1.0 LANGUAGES C CXX)

include(CTest)
enable_testing()

find_package( OpenCV REQUIRED)
include_directories( ${OpenCV_INCLUDE_DIRS} )

add_executable(testgpucpp main.cpp)

target_link_libraries( testgpucpp ${Opencv_LIBS} )

set(CPACK_PROJECT_NAME ${PROJECT_NAME})
set(CPACK_PROJECT_VERSION ${PROJECT_VERSION})
include(CPack)



# main.cpp example

#include <opencv2/opencv.hpp> // Include the main OpenCV header
#include <iostream>           // Include for std::cout
#include <windows.h>
#include <windows.h>

int main() {

    wchar_t buffer[MAX_PATH];
    // Get the current working directory using the Unicode version
    DWORD length = GetCurrentDirectoryW(MAX_PATH, buffer);
    if (length > 0 && length <= MAX_PATH) {
        std::wcout << L"Current working directory: " << buffer << std::endl;
    }
    else {
        std::cerr << "Error getting current working directory" << std::endl;
    }

    std::string imgPath = "lenna.jpg";
    // Read the image from the provided path
    cv::Mat image = cv::imread(imgPath, cv::IMREAD_COLOR);

    // Check for failure
    if (image.empty()) {
        std::cout << "Could not open or find the image at " << imgPath << std::endl;
        return -1;
    }

    // Display the image
     cv::imshow("Display window", image);
    // Wait for a keystroke in the window
    cv::waitKey(0);

    return 0;
}
