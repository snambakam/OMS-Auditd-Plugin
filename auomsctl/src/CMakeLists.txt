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

add_executable(auomsctl
    auomsctl.cpp
)
target_include_directories(auomsctl PUBLIC 
    ../include 
    ../../common/include
    ${GENERATED_INCLUDE_DIR}
)
target_link_libraries(auomsctl PRIVATE common)

if(CMAKE_BUILD_TYPE MATCHES RelWithDebInfo)
    ADD_CUSTOM_COMMAND(TARGET auomsctl POST_BUILD
        COMMAND ${CMAKE_OBJCOPY} --only-keep-debug $<TARGET_FILE:auomsctl> ${CMAKE_BINARY_DIR}/auomsctl.dbg
        COMMAND ${CMAKE_STRIP} -g $<TARGET_FILE:auomsctl>
        COMMAND ${CMAKE_OBJCOPY} --add-gnu-debuglink=auomsctl.dbg $<TARGET_FILE:auomsctl>
    )
    install(FILES ${CMAKE_BINARY_DIR}/auomsctl.dbg DESTINATION ${CMAKE_BINARY_DIR}/release/bin)
endif(CMAKE_BUILD_TYPE MATCHES RelWithDebInfo)

install(TARGETS
    auomsctl
    RUNTIME DESTINATION bin
)