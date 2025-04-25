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

add_executable(auomscollect
    auomscollect.cpp
    FileWatcher.cpp
    SchedPriority.cpp
    SPSCDataQueue.cpp
)
target_include_directories(auomscollect PUBLIC 
    ../include 
    ../../common/include
    ${GENERATED_INCLUDE_DIR}
)
target_compile_options(auomscollect PRIVATE "-Werror")
target_link_libraries(auomscollect PRIVATE common)

#
# Testing
#

add_executable(SPSCDataQueueTests
    SPSCDataQueue.cpp
    SPSCDataQueueTests.cpp
)
target_include_directories(SPSCDataQueueTests PUBLIC
    ../include
    ../../common/include
    ${GENERATED_INCLUDE_DIR}
)

if(NOT DO_STATIC_LINK)
  target_compile_definitions(SPSCDataQueueTests PUBLIC BOOST_TEST_DYN_LINK=1)
endif()

target_link_libraries(SPSCDataQueueTests
    common
    ${Boost_LIBRARIES}
    pthread
)

add_test(SPSCDataQueueTests ${CMAKE_BINARY_DIR}/SPSCDataQueueTests
    --log_sink=SPSCDataQueueTests.log
    --report_sink=SPSCDataQueueTests.report
)

if(CMAKE_BUILD_TYPE MATCHES RelWithDebInfo)
    ADD_CUSTOM_COMMAND(TARGET auomscollect POST_BUILD
        COMMAND ${CMAKE_OBJCOPY} --only-keep-debug $<TARGET_FILE:auomscollect> ${CMAKE_BINARY_DIR}/auomscollect.dbg
        COMMAND ${CMAKE_STRIP} -g $<TARGET_FILE:auomscollect>
        COMMAND ${CMAKE_OBJCOPY} --add-gnu-debuglink=auomscollect.dbg $<TARGET_FILE:auomscollect>
    )
    install(FILES ${CMAKE_BINARY_DIR}/auomscollect.dbg DESTINATION ${CMAKE_BINARY_DIR}/release/bin)
endif(CMAKE_BUILD_TYPE MATCHES RelWithDebInfo)

install(TARGETS
    auomscollect
    RUNTIME DESTINATION bin
)
