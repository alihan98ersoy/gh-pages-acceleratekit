---
title: Integrating the HMS Core SDK
description: 20
---

## Integrating the HMS Core SDK

**1. Create an Android Studio project.**
   Create a project named example in Android Studio, select Native C++, and use default settings in follow-up steps.



<img style="width: 500.00px ; padding: 5px" src="https://raw.githubusercontent.com/alihan98ersoy/gh-pages-acceleratekit/gh-pages/assets/1a.png">

**2. Add the NDK build information.**
	Add the following lines for NDK build to the /app/build.gradle file in the project directory:

```
arguments "-DANDROID_STL=c++_shared"
abiFilters "arm64-v8a","armeabi-v7a"
```

<img style="width: 500.00px ; padding: 5px" src="https://raw.githubusercontent.com/alihan98ersoy/gh-pages-acceleratekit/gh-pages/assets/2a.png">

**3. Copy libraries and header files.**
a) Download the SDK package from [SDK Download](https://developer.huawei.com/consumer/en/doc/development/HMSCore-Library/sdk-download-0000001051060752) and decompress the package.
b) Copy the header files in the SDK to the resource library.

In the /app directory in the Android Studio project, create an include folder. Copy the files in the /include directory of the SDK to the newly created include folder.

<img style="width: 500.00px ; padding: 5px" src="https://raw.githubusercontent.com/alihan98ersoy/gh-pages-acceleratekit/gh-pages/assets/3a.png">

c) Copy the .so files in the SDK to the resource library.

Create a libs folder in the /app directory and create an arm64-v8a folder and an armeabi-v7a folder in the /app/libs directory.

Copy libdispatch.so and libBlockRuntime.so in th/lib64** 4 directory of the SDK to th/libs/arm64-v8a8a directory of Android Studio.

Copy the libdispatch.so and libBlockRuntime.so files in the /lib directory of the SDK to the /libs/armeabi-v7a directory of Android Studio.

<img style="width: 500.00px ; padding: 5px" src="https://raw.githubusercontent.com/alihan98ersoy/gh-pages-acceleratekit/gh-pages/assets/3b.png">

d) Modify the CMakeLists.txt file in the app/src/main/cpp directory as follows.


```js
	# For more information about using CMake with Android Studio, read the
	# documentation: https://d.android.com/studio/projects/add-native-code.html
	# Sets the minimum version of CMake required to build the native library.
	cmake_minimum_required(VERSION 3.4.1)
	# Creates and names a library, sets it as either STATIC
	# or SHARED, and provides the relative paths to its source code.
	# You can define multiple libraries, and CMake builds them for you.
	# Gradle automatically packages shared libraries with your APK.
	add_library( # Sets the name of the library.
			native-lib
			# Sets the library as a shared library.
			SHARED
			# Provides a relative path to your source file(s).
			native-lib.cpp )
	# Searches for a specified prebuilt library and stores the path as a
	# variable. Because CMake includes system libraries in the search path by
	# default, you only need to specify the name of the public NDK library
	# you want to add. CMake verifies that the library exists before
	# completing its build.
	target_include_directories(
			native-lib
			PRIVATE
			${CMAKE_SOURCE_DIR}/../../../include)
	find_library( # Sets the name of the path variable.
			log-lib
			# Specifies the name of the NDK library that
			# you want CMake to locate.
			log )
	add_library(
			dispatch
			SHARED
			IMPORTED)
	set_target_properties(
	dispatch
			PROPERTIES IMPORTED_LOCATION
		   ${CMAKE_SOURCE_DIR}/../../../libs/${ANDROID_ABI}/libdispatch.so)
	add_library(
			BlocksRuntime
			SHARED
			IMPORTED)
	set_target_properties(
			BlocksRuntime
			PROPERTIES IMPORTED_LOCATION
			${CMAKE_SOURCE_DIR}/../../../libs/${ANDROID_ABI}/libBlocksRuntime.so)
	add_custom_command(
			TARGET native-lib POST_BUILD
			COMMAND
			${CMAKE_COMMAND} -E copy
			${CMAKE_SOURCE_DIR}/../../../libs/${ANDROID_ABI}/libdispatch.so
			${CMAKE_LIBRARY_OUTPUT_DIRECTORY}/
			COMMAND
			${CMAKE_COMMAND} -E copy
			${CMAKE_SOURCE_DIR}/../../../libs/${ANDROID_ABI}/libBlocksRuntime.so
			${CMAKE_LIBRARY_OUTPUT_DIRECTORY}/
	)
	target_compile_options(native-lib PRIVATE -fblocks)
	# Specifies libraries CMake should link to your target library. You
	# can link multiple libraries, such as libraries you define in this
	# build script, prebuilt third-party libraries, or system libraries.
	target_link_libraries( # Specifies the target library.
			native-lib
			dispatch
			BlocksRuntime
			# Links the target library to the log library
			# included in the NDK.
	${log-lib} )
```

The preceding information is added and modified. Related parameters are described as below:

•	target_include_directories: adds the directory of header files of the multi-thread library.

•	add_library: adds the .so files related to the multi-thread library.
•	set_target_properties: specifies the path of the .so file related to the multi-thread library.
•	add_custom_command: adds the command for copying .so files.
•	target_compile_option: adds the -fblocks compilation option.
•	target_link_libraries: adds the dependency of the .so files.