# cmake_minimum_required(VERSION 3.9)

project(LinVst)

INCLUDE(FindPackageHandleStandardArgs)

message( "Configuring Project LinVst.")

# Compile Flags.
################
SET(STATIC_LINKING FALSE )


# Project Path.
###############
set( CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/lib )
set( CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/lib )
set( CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/bin )

link_directories( ${CMAKE_LIBRARY_OUTPUT_DIRECTORY} )
include_directories( ${CMAKE_CURRENT_SOURCE_DIR} )

# Find wineg++, use default system or look at winehq deve package installed in /opt.

find_program (
          WINEGPP_BIN
          NAMES wineg++
          PATHS /usr/bin /opt/wine-stable/bin /opt/wine-devel/bin /opt/wine-staging/bin
)

if( NOT WINEGPP_BIN )
  message( "ERROR, cannot find wineg++." )
else()
  message( "Compiling exe with " ${WINEGPP_BIN} " compiler." )
endif()      
          
set( CMAKE_CXX_COMPILER ${WINEGPP_BIN} )

# Find wine windows header.

find_path( WIN_INCLUDE windows.h 
  /usr/include/wine-development/windows
  /usr/include/wine-development/wine/windows
  /usr/include/wine/wine/windows
  /usr/include/wine/windows
  /opt/wine-stable/include/wine/windows
  /opt/wine-devel/include/wine/windows
  /opt/wine-staging/include/wine/windows
)

if( NOT WIN_INCLUDE )
  message( "ERROR, cannot find wine windows include path (windows.h)." )
else()
  message( "Found windows headers int " ${WIN_INCLUDE} )
endif()      


set( LINVST_SVR_SRC
  ../../../lin-vst-server.cpp
  ../../../remotepluginserver.cpp
  ../../../paths.cpp
  )

set( CMAKE_CXX_STANDARD 14 )

include_directories( ${WIN_INCLUDE} )

add_executable( lin-vst-servertrack.exe ${LINVST_SVR_SRC} )
set_target_properties(  lin-vst-servertrack.exe PROPERTIES COMPILE_FLAGS " ${CMAKE_CXX_FLAGS} -Wl,-L/opt/wine-stable/lib64/wine -L/opt/wine-devel/lib64/wine -L/opt/wine-staging/lib64/wine -L/usr/lib/x86_64-linux-gnu/wine-development -m64 -fPIC -O2 -DEMBED -DEMBEDDRAG -DWAVES -DTRACKTIONWM -DVESTIGE -DNEWTIME -DINOUTMEM -DCHUNKBUF -DEMBEDRESIZE" )
set_target_properties( lin-vst-servertrack.exe PROPERTIES LINK_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -L/opt/wine-stable/lib64/wine -L/opt/wine-devel/lib64/wine -L/opt/wine-staging/lib64/wine -L/usr/lib/x86_64-linux-gnu/wine-development" )
target_link_libraries( lin-vst-servertrack.exe pthread rt )

install( TARGETS
  lin-vst-servertrack.exe  
  DESTINATION bin )

install(FILES  ${CMAKE_CURRENT_BINARY_DIR}/bin/lin-vst-servertrack.exe.so  DESTINATION bin)
