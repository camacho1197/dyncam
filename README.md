# DynCam

Application to test out reactive programming in combination with depth cameras like the Microsoft Kinect v2.

## Prerequisities

* A C++14 capable compiler
* [CMake](https://cmake.org/download/)

## Dependencies

* [C++React](https://github.com/schlangster/cpp.react) - A reactive programming library for C++11
* [Intel TBB](https://www.threadingbuildingblocks.org/) - Dependency for C++React
* [OpenCV](https://opencv.org/) - Used for image matrices and serialization
* [Qt5](https://www.qt.io/developers/) - For the GUI
* [freenect2](https://github.com/OpenKinect/libfreenect2) - For Kinect v2 support
* [glfw3](http://www.glfw.org/docs/latest/) - Dependency for freenect2
* [libturbojpeg](http://libjpeg-turbo.virtualgl.org/) - Dependency for freenect2
* [Azure Kinect](https://github.com/Microsoft/Azure-Kinect-Sensor-SDK) - For Microsoft Azure Kinect Sensor support
* [Structure.IO](https://developer.structure.io/sdk) - For Structure.IO support
* [gtest](https://github.com/google/googletest) - google test framework

If you are using Linux, most of these can be installed via your distro's package manager.

## Building

This project uses cmake and can be build like other CMake-projects. However, proper find-modules for freenect2 and C++React are still missing. You have to specify the paths yourself like below (yours paths may vary):

```
$ mkdir build
$ cd build
$ cmake .. -Dfreenect2_DIR=~/Installs/lib/cmake/freenect2/ -Dcppreact_INCLUDE_DIR=~/cpp.react/include -Dcppreact_LIBRARY=~/cpp.react/build/lib/libCppReact.a
```
For future reference, find-modules should be shipped in /cmake/modules (e.g. FindCppReact.cmake and FindFreenect2.cmake).

## Building (Windows 10)
	
- Install dependencies

    * [Visual Studio 2017](https://www.visualstudio.com/downloads/) - Make sure to install [Visual C++ Build Tools] and ATL/MFC(https://blogs.msdn.microsoft.com/vcblog/2016/11/16/introducing-the-visual-studio-build-tools/) for CMake and Windows 10 SDK.
    * [CMake for Windows](https://cmake.org/download/) - Used to build the project and other dependencies
    * [Git Bash](https://git-scm.com/download/win)
	* [Vcpkg](https://github.com/Microsoft/vcpkg) - Package manager to easily install and manage additional libraries. Just follow the given instructions
	
- Install additional dependencies using Vcpkg
	* install boost, qt5, tbb, opencv, eigen3, glfw3, libfreenect2, ffmpeg, lz4, zstd, azure-kinect-sensor-sdk and cereal with commands like in the following example
	
```
 .\vcpkg install boost:x64-windows

```
	
- Update environment variables
    - Add the following to the systems path environment variable:
    * [vcpkg dir]\installed\x64-windows\bin
	
- Clone the dyncam repo and update submodules like in the following example   

```
git clone https://gitlab.informatik.uni-bremen.de/cgvr/dyncam
cd ./dyncam
git submodule update --recursive --init
mkdir build && cd build

```

- (For using Kinect V2) Install the libusbK driver according to the windows instructions from [libfreenect2](https://github.com/OpenKinect/libfreenect2) and restart the pc

- CMake configurations
    - Open CMake and set Source Directory to dyncam's root directory 
    - Set build directory to the one we created using git bash
	- Click Configure. Make sure you use Visual Studio 15 2017 Win64 generator and to specify a toolchain file.
	- Set the path for the toolchain file to: $vcpkg dir$/scripts/buildsystems/vcpkg.cmake
    - (Uncheck build test, example, benchmark)
	- (Check build interface for using the Unreal Engine interface)
	- (Set the path for the CMAKE_INSTALL_PREFIX to DynCams root directory in your main application, e.g.: $UE project root folder$/Plugins/DynCam)
    - Click Generate and open the Visual Studio Solution.  

- Build the project called DynCam. For cereal-related compilation errors, add the following defines where needed: 

```
#ifdef max
#undef max
#endif
#ifdef min
#undef min
#endif 
```

For compression testing:
- in CMake, do check build tests before generating 
- in visual studio, build all targets/projects or at least testMain
- copy the contents from the testdata folder into "$project dir$\build\src\Tests" 
- set testMain as startProject and run it
- all compression test should run through (and take a while). The results will be encoded in a file called "resultsData.txt"
- the test code for the compression is located in "depthcompression_test.cpp", the output format can be seen in the function "WriteResults()" 

For using the debug mode with Kinect V2:
- In Freenect2Cam.cpp set 'serial_' manually, e.g.: change 'serial_ = "002742663447";' to the spedific serial number of your kinect V2
- Copy the freenect2d.dll from $vcpkg dir$\installed\x64-windows\debug\bin to the projects executable folder and delete the 'd' in the name
- Do the same with the lz4d.dll
- Start in the IDE and add a breakpoint in Freenect2.cpp,'Open()'-function to the line 'if (state != State::DEVICE_OPEN && Connect())' and step over it before clicking continue

## Contributing

We are using a modified version of the [Gitflow Workflow](https://www.atlassian.com/git/tutorials/comparing-workflows#gitflow-workflow), where new features and fixes will be developed on separate branches (e.g. feature/xyz; fix/issue#001 etc.) and then merged back into *master*, which serves as the main development branch. Occasianlly, stable states of the programme will be merged onto the *stable* branch and tagged with a version number. The stable state must always build correctly and run without deal-breaking bugs. Features **may not** be merged directly into *stable*, only hotfixes for severe bugs may be merged directly. If a hotfix was applied, make sure to also merge it into *master*, to keep the development state up-to-date.

Note that in contrast to the regular Gitflow conecpt, *master* => *stable* and *develop* => *master*.