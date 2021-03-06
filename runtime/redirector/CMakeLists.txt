################################################################################
# Copyright (c) 2017, 2020 IBM Corp. and others
#
# This program and the accompanying materials are made available under
# the terms of the Eclipse Public License 2.0 which accompanies this
# distribution and is available at https://www.eclipse.org/legal/epl-2.0/
# or the Apache License, Version 2.0 which accompanies this distribution and
# is available at https://www.apache.org/licenses/LICENSE-2.0.
#
# This Source Code may also be made available under the following
# Secondary Licenses when the conditions for such availability set
# forth in the Eclipse Public License, v. 2.0 are satisfied: GNU
# General Public License, version 2 with the GNU Classpath
# Exception [1] and GNU General Public License, version 2 with the
# OpenJDK Assembly Exception [2].
#
# [1] https://www.gnu.org/software/classpath/license.html
# [2] http://openjdk.java.net/legal/assembly-exception.html
#
# SPDX-License-Identifier: EPL-2.0 OR Apache-2.0 OR GPL-2.0 WITH Classpath-exception-2.0 OR LicenseRef-GPL-2.0 WITH Assembly-exception
################################################################################

set(OMR_ENHANCED_WARNINGS OFF)

set_source_files_properties("${CMAKE_CURRENT_BINARY_DIR}/generated.c" PROPERTIES GENERATED TRUE)

################################################################################
# Redirector
################################################################################
j9vm_add_library(jvm_redirect
	SHARED
	OUTPUT_NAME jvm

	"${CMAKE_CURRENT_BINARY_DIR}/generated.c"
	redirector.c
)
target_link_libraries(jvm_redirect
	PRIVATE
		j9vm_interface
		${CMAKE_DL_LIBS}
)
target_include_directories(jvm_redirect
	PRIVATE
		"${j9vm_SOURCE_DIR}/j9vm"
		"${j9vm_BINARY_DIR}/j9vm"
)
if(OMR_OS_ZOS)
	target_include_directories(jvm_redirect
		PRIVATE
			"${omr_SOURCE_DIR}/port/zos390"
	)
	# redirector needs getUserExtendedPrivateAreaMemoryType from omrutil,
	# but we don't want to introduce any transitive dependencies, so we use $<TARGET_FILE>.
	target_link_libraries(jvm_redirect PRIVATE $<TARGET_FILE:omrutil>)

	# Ensure that omrutil is built before we attempt to link it.
	add_dependencies(jvm_redirect omrutil)
endif()
set_target_properties(jvm_redirect PROPERTIES
	ARCHIVE_OUTPUT_NAME redirector_jvm
	LIBRARY_OUTPUT_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}"
	RUNTIME_OUTPUT_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}"
)

# The redirector should have the same exports as the jvm
get_target_property(jvm_exports jvm EXPORTED_SYMBOLS)
omr_add_exports(jvm_redirect ${jvm_exports})

install(
	TARGETS jvm_redirect
	LIBRARY DESTINATION ${j9vm_SOURCE_DIR}/redirector
	RUNTIME DESTINATION ${j9vm_SOURCE_DIR}/redirector
)
