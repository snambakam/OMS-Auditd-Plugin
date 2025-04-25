####
# microsoft-oms-auditd-plugin
#
# Copyright (c) Microsoft Corporation
#
# All rights reserved. 
#
# MIT License
#
# Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the ""Software""), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED *AS IS*, WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
####

cmake_minimum_required(VERSION 2.8.12)
project(auoms)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++17")
set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -ggdb")
set(CMAKE_CXX_FLAGS_RELWITHDEBINFO "${CMAKE_CXX_FLAGS_RELWITHDEBINFO} -ggdb -Wl,-z,relro -Wl,-z,now -fstack-protector-strong -D_FORTIFY_SOURCE=2")
set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} -ggdb")
set(CMAKE_C_FLAGS_RELWITHDEBINFO "${CMAKE_C_FLAGS_RELWITHDEBINFO} -ggdb -Wl,-z,relro -Wl,-z,now -fstack-protector-strong -D_FORTIFY_SOURCE=2")

if(DO_STATIC_LINK)
    # See https://gcc.gnu.org/onlinedocs/libstdc++/manual/license.html
    # and https://www.gnu.org/licenses/gcc-exception-3.1-faq.en.html
    # for why it is safe to use -static-libgcc -static-libstdc++
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -Wl,-z,relro -Wl,-z,now -static-libgcc -static-libstdc++ -Wl,--no-as-needed -lrt -Wl,--as-needed")
    set(RE2_LIBS "libre2.a")
else()
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS}")
    set(RE2_LIBS "re2")
endif()

if (DEPS_INCLUDE)
    include_directories(BEFORE ${DEPS_INCLUDE})
endif()

if (DEPS_LIB)
    link_directories(BEFORE ${DEPS_LIB})
endif()

if (NOT ENV_CONFIG_PATH)
    set(ENV_CONFIG_PATH ${CMAKE_SOURCE_DIR}/build/env_config.h)
endif()

if (NO_EXTRA_INTERP_PROTO)
    add_compile_definitions(NO_INTERP_EXTRA_PROTO)
endif()

set(GENERATED_INCLUDE_DIR ${CMAKE_BINARY_DIR}/generated/include)
file(MAKE_DIRECTORY ${GENERATED_INCLUDE_DIR})

#Setup CMake to run tests
enable_testing()

if(DO_STATIC_LINK)
  set(Boost_USE_STATIC_LIBS ON)
endif()
find_package(Boost 1.48 COMPONENTS unit_test_framework REQUIRED)
include_directories (${Boost_INCLUDE_DIRS})

add_subdirectory(common/src)
add_subdirectory(auoms/src)
add_subdirectory(auomsctl/src)
add_subdirectory(auomscollect/src)
add_subdirectory(file2sock/src)
add_subdirectory(fakeaudispd/src)
add_subdirectory(testreceiver/src)

include(GNUInstallDirs)

install(FILES
  LICENSE
  THIRD_PARTY_IP_NOTICE
  DESTINATION ${CMAKE_INSTALL_PREFIX}
)
install(FILES
  installer/conf/auoms.conf
  installer/conf/auomscollect.conf
  installer/conf/example_output.conf
  installer/conf/example_redact.conf
  DESTINATION ${CMAKE_INSTALL_SYSCONFDIR}
)
install(FILES
  installer/auoms.init
  installer/auoms.service
  DESTINATION ${CMAKE_INSTALL_PREFIX}
)
install(FILES
  installer/selinux/auoms.fc
  installer/selinux/auoms.te
  DESTINATION ${CMAKE_INSTALL_DATADIR}
)

#
# Get Project version
#
#file(STRINGS "${CMAKE_SOURCE_DIR}/build/auoms.version" VERSION_LINES)
#foreach(line IN LISTS VERSION_LINES)
#    if(line MATCHES "^AUOMS_BUILDVERSION_MAJOR=([0-9]+)")
#        set(AUOMS_VERSION_MAJOR "${CMAKE_MATCH_1}")
#    elseif(line MATCHES "^AUOMS_BUILDVERSION_MINOR=([0-9]+)")
#        set(AUOMS_VERSION_MINOR "${CMAKE_MATCH_1}")
#    elseif(line MATCHES "^AUOMS_BUILDVERSION_PATCH=([0-9]+)")
#        set(AUOMS_VERSION_PATCH "${CMAKE_MATCH_1}")
#    elseif(line MATCHES "^AUOMS_BUILDVERSION_BUILDNR=([0-9]+)")
#        set(AUOMS_VERSION_BUILDNR "${CMAKE_MATCH_1}")
#    endif()
#endforeach()
#
#set(AUOMS_VERSION 
#  "${AUOMS_VERSION_MAJOR}.${AUOMS_VERSION_MINOR}.${AUOMS_VERSION_PATCH}"
#)
