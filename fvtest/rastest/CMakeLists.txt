###############################################################################
# Copyright (c) 2017, 2017 IBM Corp. and others
#
# This program and the accompanying materials are made available under
# the terms of the Eclipse Public License 2.0 which accompanies this
# distribution and is available at http://eclipse.org/legal/epl-2.0
# or the Apache License, Version 2.0 which accompanies this distribution
# and is available at https://www.apache.org/licenses/LICENSE-2.0.
#
# This Source Code may also be made available under the following Secondary
# Licenses when the conditions for such availability set forth in the
# Eclipse Public License, v. 2.0 are satisfied: GNU General Public License,
# version 2 with the GNU Classpath Exception [1] and GNU General Public
# License, version 2 with the OpenJDK Assembly Exception [2].
#
# [1] https://www.gnu.org/software/classpath/license.html
# [2] http://openjdk.java.net/legal/assembly-exception.html
#
# SPDX-License-Identifier: EPL-2.0 OR Apache-2.0 OR GPL-2.0 WITH Classpath-exception-2.0 OR LicenseRef-GPL-2.0 WITH Assembly-exception
###############################################################################

add_tracegen(omr_test.tdf)
include_directories(${CMAKE_CURRENT_BINARY_DIR})

add_library(bindthreadagent SHARED
	bindthreadagent.c
	omragent.def
)
target_link_libraries(bindthreadagent PUBLIC omrtestutil)

add_library(memorycategoriesagent SHARED
	memorycategoriesagent.c
	omragent.def
)
target_link_libraries(memorycategoriesagent PRIVATE omrtestutil)

add_library(traceagent SHARED
	traceagent.c
	omragent.def
)
target_link_libraries(traceagent PRIVATE omrtestutil)

add_library(traceNotStartedAgent SHARED
	traceNotStartedAgent.c
	omragent.def
)
target_link_libraries(traceNotStartedAgent PRIVATE omrtestutil)



add_library(cpuLoadAgent SHARED
	cpuLoadAgent.c
	omragent.def
)
target_link_libraries(cpuLoadAgent PUBLIC omrtestutil)

add_library(invalidAgentMissingOnLoad SHARED
	invalidAgentMissingOnLoad.c
	invalidAgentMissingOnLoad.def
)

add_library(invalidAgentMissingOnUnload SHARED
	invalidAgentMissingOnUnload.c
	invalidAgentMissingOnUnload.def
)
add_library(invalidAgentReturnError SHARED
	invalidAgentReturnError.c
	omragent.def
)

add_library(sampleSubscriber SHARED
	sampleSubscriber.c
	omragent.def
)

add_library(subscriberAgent SHARED
	subscriberAgent.c
	omragent.def
)
target_link_libraries(subscriberAgent PRIVATE omrtestutil)

add_library(subscriberAgentWithJ9thread SHARED
	subscriberAgentWithJ9thread.c
	omragent.def
)
target_link_libraries(subscriberAgentWithJ9thread PRIVATE omrtestutil)

add_library(traceOptionAgent SHARED
	traceOptionAgent.c
	omragent.def
)


add_executable(omrrastest
	agentNegativeTest.cpp
	agentTest.cpp
	main.cpp
	memoryCategoriesTest.cpp
	methodDictionaryTest.cpp
	rasTestHelpers.cpp
	traceLifecycleTest.cpp
	traceLogTest.cpp
	traceRecordHelpers.cpp
	traceTest.cpp
	ut_omr_test.c
)
target_link_libraries(omrrastest
	PRIVATE
		omrGtestGlue
		omrtestutil
		omrport
		j9thrstatic
		omrcore
		omrgc
)
#  With MSVC Rastest can stack overflow with default stack size
# depending on debugging and optimization settings
if(CMAKE_C_COMPILER_ID STREQUAL "MSVC")
	math(EXPR new_stack_size "8 * 1024 * 1024")
	set_property(TARGET omrrastest APPEND PROPERTY LINK_FLAGS " /STACK:${new_stack_size}")
endif()
add_dependencies(omrrastest
	bindthreadagent
	cpuLoadAgent
	invalidAgentMissingOnLoad
	invalidAgentReturnError
	memorycategoriesagent
	sampleSubscriber
	traceagent
	traceNotStartedAgent
)

add_executable(omrsubscribertest
	main.cpp
	rasTestHelpers.cpp
	subscriberTest.cpp
	ut_omr_test.c
)
if(OMR_THR_FORK_SUPPORT)
	target_sources(omrsubscribertest PRIVATE subscriberForkTest.cpp)
endif()
target_link_libraries(omrsubscribertest
	PRIVATE
		omrGtestGlue
		omrtestutil
		j9thrstatic
		omrcore
		omrtrace
		omrport
		omrgc
)
add_dependencies(omrsubscribertest
	subscriberAgent
	subscriberAgentWithJ9thread
)

add_executable(omrtraceoptiontest
	main.cpp
	rasTestHelpers.cpp
	traceOptionTest.cpp
	ut_omr_test.c
)
target_link_libraries(omrtraceoptiontest
	PRIVATE
		omrGtestGlue
		omrtestutil
		j9thrstatic
		omrcore
		omrport
		omrgc
)
add_dependencies(omrtraceoptiontest
	traceOptionAgent
)

add_test(NAME rastest COMMAND omrrastest)
add_test(NAME subscribertest COMMAND omrsubscribertest)
add_test(NAME traceoptiontest COMMAND omrtraceoptiontest)

set_target_properties(
	omrrastest
	omrsubscribertest
	omrtraceoptiontest
	PROPERTIES
	FOLDER "fvtest/rastest"
)

set_target_properties(
	bindthreadagent
	cpuLoadAgent
	invalidAgentMissingOnLoad
	invalidAgentMissingOnUnload
	invalidAgentReturnError
	memorycategoriesagent
	sampleSubscriber
	subscriberAgent
	subscriberAgentWithJ9thread
	traceagent
	traceNotStartedAgent
	traceOptionAgent

	PROPERTIES
	FOLDER "fvtest/rastest/agents"
)
