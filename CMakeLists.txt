cmake_minimum_required (VERSION 2.6)
project (AMDTOSAPIWrappers)

# The version number.
set (AMDTOSAPIWrappers_VERSION_MAJOR 1)
set (AMDTOSAPIWrappers_VERSION_MINOR 0)

# project-specific preprocessor directives
add_definitions(-DAMDTOSAPIWRAPPERS_EXPORTS)

# Windows-specific preprocessor directives
if(${CMAKE_SYSTEM_NAME} STREQUAL "Windows")
    add_definitions(-DDBGHELP_TRANSLATE_TCHAR -D_CRT_SECURE_NO_WARNINGS)
endif()

# Add this definitions if static linkage is used with the other projects
add_definitions(-DAMDTBASETOOLS_STATIC -DAMDTOSWRAPPERS_STATIC)
 
# configure a header file to pass some of the CMake settings
# to the source code
configure_file (
  "${PROJECT_SOURCE_DIR}/AMDTOSAPIWrappersConfig.h.in"
  "${PROJECT_BINARY_DIR}/AMDTOSAPIWrappersConfig.h"
  ) 

# add the binary tree to the search path for include files
# so that we will find ProjectConfig.h
include_directories("${PROJECT_SOURCE_DIR}/Include")
include_directories("${PROJECT_SOURCE_DIR}/src")
include_directories("${PROJECT_SOURCE_DIR}/../Miniz")
include_directories("${PROJECT_SOURCE_DIR}/../ADLUtil")
include_directories("${PROJECT_SOURCE_DIR}/../AMDTMutex")
include_directories("${PROJECT_SOURCE_DIR}/../TSingleton")
include_directories("${PROJECT_SOURCE_DIR}/../")
include_directories("${PROJECT_SOURCE_DIR}/../../Lib/Ext/tinyxml/2.6.2")
include_directories(${PROJECT_SOURCE_DIR}/../../Lib/Ext/zlib/1.2.8)
include_directories(${PROJECT_SOURCE_DIR}/../../Lib/AMD/APPSDK/3-0/include)
include_directories("${PROJECT_SOURCE_DIR}/../../Lib/AMD/ADL/include")

# Source files that are common to Windows and Linux
file(GLOB COMMON_SRC
    "src/common/*.h"
    "src/common/*.cpp"
)

# Windows-specific source files
file(GLOB WINDOWS_SRC
    "src/win32/*.h"
    "src/win32/*.cpp"
)

# Linux-specific source files
file(GLOB LINUX_SRC
    "src/linux/*.h"
    "src/linux/*.cpp"
)

# add the library (change STATIC to SHARED for dynamic library build)
# Pick up the source files that are relevant to the platform
if(${CMAKE_SYSTEM_NAME} STREQUAL "Windows")
    add_library(AMDTOSAPIWrappers STATIC ${COMMON_SRC} ${WINDOWS_SRC})
    find_library(ZLIB_LIBRARY zlibstat HINTS "${PROJECT_SOURCE_DIR}/../../Lib/Ext/zlib/1.2.8/contrib/vstudio/vc14/x86/ZlibStatRelease")
    target_link_libraries (AMDTOSAPIWrappers ${ZLIB_LIBRARY})
elseif(${CMAKE_SYSTEM_NAME} STREQUAL "Linux")
    include_directories("/usr/include/cairo")
    include_directories("/usr/include/pango-1.0")
    include_directories("/usr/include/atk-1.0")
    include_directories("/usr/include/gtk-2.0")
    include_directories("/usr/include/gdk-pixbuf-2.0")
    include_directories("/usr/include/glib-2.0")
    include_directories("/usr/lib64/gtk-2.0/include")
    include_directories("/usr/lib64/glib-2.0/include/")
    include_directories("/usr/lib/x86_64-linux-gnu/glib-2.0/include")
    include_directories("/usr/lib/x86_64-linux-gnu/gtk-2.0/include")
    include_directories("/usr/include/x86_64-linux-gnu/")
    find_package (Threads)
    add_library(AMDTOSAPIWrappers STATIC ${COMMON_SRC} ${LINUX_SRC})
endif()

# set binary suffix
set_target_properties(AMDTOSAPIWrappers PROPERTIES DEBUG_POSTFIX -d)

# Link settings
target_link_libraries (AMDTOSAPIWrappers AMDTOSWrappers AMDTBaseTools ${CMAKE_DL_LIBS} ${CMAKE_THREAD_LIBS_INIT})

