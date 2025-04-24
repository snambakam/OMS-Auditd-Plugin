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

add_executable(auoms
    AuditRulesMonitor.cpp
    auoms.cpp
    CollectionMonitor.cpp
    EventFilter.cpp
    Inputs.cpp
    Outputs.cpp
    SyscallMetrics.cpp
    SystemMetrics.cpp
)
target_include_directories(auoms PUBLIC 
    ../include 
    ../../common/include
    ${GENERATED_INCLUDE_DIR}
)
target_link_libraries(auoms PRIVATE common)

#
# Testing
#

add_executable(OutputInputTests
        OutputInputTests.cpp
        Inputs.cpp
)
target_include_directories(OutputInputTests PUBLIC
    ../include 
    ../../common/include
    ${GENERATED_INCLUDE_DIR}
)

if(NOT DO_STATIC_LINK)
  target_compile_definitions(OutputInputTests PUBLIC BOOST_TEST_DYN_LINK=1)
endif()

target_link_libraries(OutputInputTests
    common
    ${Boost_LIBRARIES}
    pthread
)

add_test(OutputInputTests
    ${CMAKE_BINARY_DIR}/OutputInputTests
    --log_sink=OutputInputTests.log
    --report_sink=OutputInputTests.report
)
