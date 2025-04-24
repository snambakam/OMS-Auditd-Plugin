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

set(GENERATED_HEADER_AUOMS_VERSION ${GENERATED_INCLUDE_DIR}/auoms_version.h)
set(AUOMS_VERSION_SOURCE ${CMAKE_SOURCE_DIR}/build/auoms.version)
add_custom_command(
        OUTPUT ${GENERATED_HEADER_AUOMS_VERSION}
        COMMAND bash -c \"grep AUOMS_BUILDVERSION ${AUOMS_VERSION_SOURCE} | head -n 4 | cut -d'=' -f2 | tr '\\n' '.' | sed 's/^\\\([0-9][0-9]*\\.[0-9][0-9]*\\.[0-9][0-9]*\\.[0-9][0-9]*\\\)\\.*/\#define AUOMS_VERSION \\"\\1\\"/' > ${GENERATED_HEADER_AUOMS_VERSION}\"
        MAIN_DEPENDENCY ${AUOMS_VERSION_SOURCE}
)
set(GENERATED_HEADER_ENV_CONFIG ${GENERATED_INCLUDE_DIR}/env_config.h)
add_custom_command(
        OUTPUT ${GENERATED_HEADER_ENV_CONFIG}
        COMMAND ln -f -s ${ENV_CONFIG_PATH} ${GENERATED_HEADER_ENV_CONFIG}
        MAIN_DEPENDENCY ${ENV_CONFIG_PATH}
)

add_library(common STATIC
    AbstractEventWriter.cpp
    AuditRules.cpp
    AuditStatus.cpp
    CGroups.cpp
    CmdlineRedactor.cpp
    Config.cpp
    CPULimits.cpp
    Event.cpp
    EventAggregator.cpp
    EventMatcher.cpp
    EventPrioritizer.cpp
    EventWriterConfig.cpp
    ExecUtil.cpp
    ExecveConverter.cpp
    FileUtils.cpp
    FiltersEngine.cpp
    FluentEventWriter.cpp
    Input.cpp
    Interpret.cpp
    IO.cpp
    KernelInfo.cpp
    LockFile.cpp
    Logger.cpp
    Metrics.cpp
    Netlink.cpp
    NetlinkAudit.cpp
    OMSEventWriter.cpp
    OperationalStatus.cpp
    Output.cpp
    PriorityQueue.cpp
    ProcessInfo.cpp
    ProcessTree.cpp
    ProcFilter.cpp
    ProcMetrics.cpp
    Queue.cpp
    RawEventAccumulator.cpp
    RawEventProcessor.cpp
    RawEventRecord.cpp
    RunBase.cpp
    Signals.cpp
    StdinReader.cpp
    StringUtils.cpp
    SyslogEventWriter.cpp
    TempDir.cpp
    TempFile.cpp
    TranslateArch.cpp
    TranslateErrno.cpp
    TranslateField.cpp
    TranslateFieldType.cpp
    TranslateRecordType.cpp
    TranslateSyscall.cpp
    UnixDomainListener.cpp
    UnixDomainWriter.cpp
    UserDB.cpp
    Version.cpp
)
target_include_directories(common PUBLIC
    ../include
    ${GENERATED_INCLUDE_DIR}
)
target_link_libraries(common
        ${RE2_LIBS}
        dl
        pthread
        rt
)
add_custom_target(generate_headers_auoms_version DEPENDS
    ${GENERATED_HEADER_AUOMS_VERSION}
)
add_custom_target(generate_headers_env_config DEPENDS
    ${GENERATED_HEADER_ENV_CONFIG}
)
add_dependencies(common
    generate_headers_auoms_version
    generate_headers_env_config
)

#
# Testing
#

###
add_executable(CmdlineRedactorTests
    CmdlineRedactorTests.cpp
    CmdlineRedactor.cpp
    Config.cpp
    Logger.cpp
    FileUtils.cpp
    StringUtils.cpp
    TempDir.cpp
)
target_include_directories(CmdlineRedactorTests PUBLIC
    ../include
)

if(NOT DO_STATIC_LINK)
  target_compile_definitions(CmdlineRedactorTests PUBLIC BOOST_TEST_DYN_LINK=1)
endif()

target_link_libraries(CmdlineRedactorTests
        ${RE2_LIBS}
        ${Boost_LIBRARIES}
        pthread
)
add_test(CmdlineRedactorTests
    ${CMAKE_BINARY_DIR}/CmdlineRedactorTests
    --log_sink=CmdlineRedactorTests.log
    --report_sink=CmdlineRedactorTests.report
)
###
add_executable(ConfigTests
    TempFile.cpp
    Config.cpp
    ConfigTests.cpp
)
target_include_directories(ConfigTests PUBLIC
    ../include
)

if(NOT DO_STATIC_LINK)
  target_compile_definitions(ConfigTests PUBLIC BOOST_TEST_DYN_LINK=1)
endif()

target_link_libraries(ConfigTests ${Boost_LIBRARIES})

add_test(ConfigTests
    ${CMAKE_BINARY_DIR}/ConfigTests
    --log_sink=ConfigTests.log
    --report_sink=ConfigTests.report
)
###
add_executable(EventTests
    TempDir.cpp
    Logger.cpp
    PriorityQueue.cpp
    FileUtils.cpp
    Event.cpp
    EventTests.cpp
)
target_include_directories(EventTests PUBLIC
    ../include
)

if(NOT DO_STATIC_LINK)
  target_compile_definitions(EventTests PUBLIC BOOST_TEST_DYN_LINK=1)
endif()

target_link_libraries(EventTests
    ${Boost_LIBRARIES}
    pthread
)

add_test(EventTests
    ${CMAKE_BINARY_DIR}/EventTests
    --log_sink=EventTests.log
    --report_sink=EventTests.report
)
###
add_executable(EventAggregatorTests
    EventAggregatorTests.cpp
    EventAggregator.cpp
    EventMatcher.cpp
    Event.cpp
    TranslateRecordType.cpp
    StringUtils.cpp
    TestEventData.cpp
    TempFile.cpp
)
target_include_directories(EventAggregatorTests PUBLIC
    ../include
    ${GENERATED_INCLUDE_DIR}
)

if(NOT DO_STATIC_LINK)
  target_compile_definitions(EventAggregatorTests PUBLIC BOOST_TEST_DYN_LINK=1)
endif()

target_link_libraries(EventAggregatorTests
    ${RE2_LIBS}
    ${Boost_LIBRARIES}
    pthread
)

add_test(EventAggregatorTests
    ${CMAKE_BINARY_DIR}/EventAggregatorTests
    --log_sink=EventAggregatorTests.log
    --report_sink=EventAggregatorTests.report
)
###
add_executable(EventMatcherTests
    EventMatcherTests.cpp
    EventMatcher.cpp
    Event.cpp
    TranslateRecordType.cpp
    StringUtils.cpp
    TestEventData.cpp
)
target_include_directories(EventMatcherTests PUBLIC
    ../include
    ${GENERATED_INCLUDE_DIR}
)

if(NOT DO_STATIC_LINK)
  target_compile_definitions(EventMatcherTests PUBLIC BOOST_TEST_DYN_LINK=1)
endif()

target_link_libraries(EventMatcherTests
    ${RE2_LIBS}
    ${Boost_LIBRARIES}
    pthread
)

add_test(EventMatcherTests
    ${CMAKE_BINARY_DIR}/EventMatcherTests
    --log_sink=EventMatcherTests.log
    --report_sink=EventMatcherTests.report
)
###
add_executable(EventProcessorTests
    Config.cpp
    CmdlineRedactor.cpp
    Event.cpp
    ExecveConverter.cpp
    EventProcessorTests.cpp
    FileUtils.cpp
    FiltersEngine.cpp
    Interpret.cpp
    Logger.cpp
    Metrics.cpp
    ProcFilter.cpp
    ProcessInfo.cpp
    ProcessTree.cpp
    RawEventAccumulator.cpp
    RawEventProcessor.cpp
    RawEventRecord.cpp
    RunBase.cpp
    Signals.cpp
    StringUtils.cpp
    TempDir.cpp
    TestEventData.cpp
    TranslateArch.cpp
    TranslateField.cpp
    TranslateFieldType.cpp
    TranslateRecordType.cpp
    TranslateSyscall.cpp
    UserDB.cpp
)
target_include_directories(EventProcessorTests PUBLIC
    ../include
    ${GENERATED_INCLUDE_DIR}
)

if(NOT DO_STATIC_LINK)
  target_compile_definitions(EventProcessorTests PUBLIC BOOST_TEST_DYN_LINK=1)
endif()

target_link_libraries(EventProcessorTests
    ${Boost_LIBRARIES}
    ${RE2_LIBS}
    dl
    pthread
    rt
)

add_test(EventProcessorTests
    ${CMAKE_BINARY_DIR}/EventProcessorTests
    --log_sink=EventProcessorTests.log
    --report_sink=EventProcessorTests.report
)
###
add_executable(ExecveConverterTests
    Event.cpp
    ExecveConverter.cpp
    ExecveConverterTests.cpp
    Logger.cpp
    Metrics.cpp
    RawEventAccumulator.cpp
    RawEventRecord.cpp
    RunBase.cpp
    StringUtils.cpp
    TranslateRecordType.cpp
)
target_include_directories(ExecveConverterTests PUBLIC
    ../include
    ${GENERATED_INCLUDE_DIR}
)

if(NOT DO_STATIC_LINK)
  target_compile_definitions(ExecveConverterTests PUBLIC BOOST_TEST_DYN_LINK=1)
endif()

target_link_libraries(ExecveConverterTests
    ${Boost_LIBRARIES}
    dl
    pthread
    rt
)

add_test(ExecveConverterTests
    ${CMAKE_BINARY_DIR}/ExecveConverterTests
    --log_sink=ExecveConverterTests.log
    --report_sink=ExecveConverterTests.report
)
###
add_executable(FluentEventWriterTests
    AbstractEventWriter.cpp
    CmdlineRedactor.cpp
    Config.cpp
    Event.cpp
    ExecveConverter.cpp
    FileUtils.cpp
    FluentEventWriter.cpp
    FluentEventWriterTests.cpp
    Logger.cpp
    StringUtils.cpp
    TestEventData.cpp
)
target_include_directories(FluentEventWriterTests PUBLIC
    ../include
    ${GENERATED_INCLUDE_DIR}
)

if(NOT DO_STATIC_LINK)
    target_compile_definitions(FluentEventWriterTests PUBLIC
        BOOST_TEST_DYN_LINK=1
    )
endif()

target_link_libraries(FluentEventWriterTests
    ${Boost_LIBRARIES}
    ${RE2_LIBS}
    pthread
)

add_test(FluentEventWriterTests
    ${CMAKE_BINARY_DIR}/FluentEventWriterTests
    --log_sink=FluentEventWriterTests.log
    --report_sink=FluentEventWriterTests.report
)
###
add_executable(OMSEventWriterTests
    AbstractEventWriter.cpp
    Config.cpp
    Event.cpp
    EventWriterConfig.cpp
    ExecveConverter.cpp
    OMSEventWriter.cpp
    OMSEventWriterTests.cpp
    Logger.cpp
    ProcFilter.cpp
    RunBase.cpp
    Signals.cpp
    StringUtils.cpp
    TestEventData.cpp
    UserDB.cpp
)
target_include_directories(OMSEventWriterTests PUBLIC
    ../include
    ${GENERATED_INCLUDE_DIR}
)

if(NOT DO_STATIC_LINK)
  target_compile_definitions(OMSEventWriterTests PUBLIC BOOST_TEST_DYN_LINK=1)
endif()

target_link_libraries(OMSEventWriterTests
    ${Boost_LIBRARIES}
    ${RE2_LIBS}
    pthread
)

add_test(OMSEventWriterTests
    ${CMAKE_BINARY_DIR}/OMSEventWriterTests
    --log_sink=OMSEventWriterTests.log
    --report_sink=OMSEventWriterTests.report
)
###
add_executable(PriorityQueueTests
    FileUtils.cpp
    Logger.cpp
    PriorityQueue.cpp
    PriorityQueueTests.cpp
    TempDir.cpp
)
target_include_directories(PriorityQueueTests PUBLIC
    ../include
)

if(NOT DO_STATIC_LINK)
  target_compile_definitions(PriorityQueueTests PUBLIC BOOST_TEST_DYN_LINK=1)
endif()

target_link_libraries(PriorityQueueTests
    ${Boost_LIBRARIES}
    pthread
)

add_test(PriorityQueueTests
    ${CMAKE_BINARY_DIR}/PriorityQueueTests
    --log_sink=PriorityQueueTests.log
    --report_sink=PriorityQueueTests.report
)
###
add_executable(ProcessInfoTests
    Config.cpp
    Event.cpp
    ExecveConverter.cpp
    FiltersEngine.cpp
    Logger.cpp
    ProcFilter.cpp
    ProcessInfo.cpp
    ProcessInfoTests.cpp
    ProcessTree.cpp
    RunBase.cpp
    Signals.cpp
    StringUtils.cpp
    UserDB.cpp
)
target_include_directories(ProcessInfoTests PUBLIC
    ../include
    ${GENERATED_INCLUDE_DIR}
)

target_link_libraries(ProcessInfoTests
    ${Boost_LIBRARIES}
    ${RE2_LIBS}
    dl
    pthread
    rt
)

if(NOT DO_STATIC_LINK)
  target_compile_definitions(ProcessInfoTests PUBLIC BOOST_TEST_DYN_LINK=1)
endif()

add_test(ProcessInfoTests
    ${CMAKE_BINARY_DIR}/ProcessInfoTests
    --log_sink=ProcessInfoTests.log
    --report_sink=ProcessInfoTests.report
)
###
add_executable(ProcessTreeTests
    Config.cpp
    Event.cpp
    ExecveConverter.cpp
    FiltersEngine.cpp
    Logger.cpp
    ProcFilter.cpp
    ProcessInfo.cpp
    ProcessTree.cpp
    ProcessTreeTests.cpp
    RunBase.cpp
    Signals.cpp
    StringUtils.cpp
    UserDB.cpp
)
target_include_directories(ProcessTreeTests PUBLIC
    ../include
    ${GENERATED_INCLUDE_DIR}
)

if(NOT DO_STATIC_LINK)
  target_compile_definitions(ProcessTreeTests PUBLIC BOOST_TEST_DYN_LINK=1)
endif()

target_link_libraries(ProcessTreeTests
    ${Boost_LIBRARIES}
    ${RE2_LIBS}
    dl
    pthread
    rt
)

add_test(ProcessTreeTests
    ${CMAKE_BINARY_DIR}/ProcessTreeTests
    --log_sink=ProcessTreeTests.log
    --report_sink=ProcessTreeTests.report
)
###
add_executable(QueueTests
    Logger.cpp
    Queue.cpp
    QueueTests.cpp
    TempFile.cpp
)
target_include_directories(QueueTests PUBLIC
    ../include
)

if(NOT DO_STATIC_LINK)
  target_compile_definitions(QueueTests PUBLIC BOOST_TEST_DYN_LINK=1)
endif()

target_link_libraries(QueueTests ${Boost_LIBRARIES})

add_test(QueueTests
    ${CMAKE_BINARY_DIR}/QueueTests
    --log_sink=QueueTests.log
    --report_sink=QueueTests.report
)
###
add_executable(StringTests
    StringUtils.cpp
    StringTests.cpp
)
target_include_directories(StringTests PUBLIC
    ../include
)

if(NOT DO_STATIC_LINK)
  target_compile_definitions(StringTests PUBLIC BOOST_TEST_DYN_LINK=1)
endif()

target_link_libraries(StringTests ${Boost_LIBRARIES})

add_test(StringTests
    ${CMAKE_BINARY_DIR}/StringTests
    --log_sink=StringTests.log
    --report_sink=StringTests.report
)
###
add_executable(UserDBTests
    Logger.cpp
    Signals.cpp
    TempDir.cpp
    UserDB.cpp
    UserDBTests.cpp
)
target_include_directories(UserDBTests PUBLIC
    ../include
)

if(NOT DO_STATIC_LINK)
  target_compile_definitions(UserDBTests PUBLIC BOOST_TEST_DYN_LINK=1)
endif()

target_link_libraries(UserDBTests ${Boost_LIBRARIES} pthread)

add_test(UserDBTests
    ${CMAKE_BINARY_DIR}/UserDBTests
    --log_sink=UserDBTests.log
    --report_sink=UserDBTests.report
)
###
