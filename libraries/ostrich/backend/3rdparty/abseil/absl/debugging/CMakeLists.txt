#
# Copyright 2017 The Abseil Authors.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

list(APPEND DEBUGGING_PUBLIC_HEADERS
  "failure_signal_handler.h"
  "leak_check.h"
  "stacktrace.h"
  "symbolize.h"
)


list(APPEND DEBUGGING_INTERNAL_HEADERS
  "internal/address_is_readable.h"
  "internal/demangle.h"
  "internal/elf_mem_image.h"
  "internal/examine_stack.h"
  "internal/stacktrace_config.h"
  "internal/symbolize.h"
  "internal/vdso_support.h"
)


list(APPEND STACKTRACE_SRC
  "stacktrace.cc"
  "internal/address_is_readable.cc"
  "internal/elf_mem_image.cc"
  "internal/vdso_support.cc"
  ${DEBUGGING_PUBLIC_HEADERS}
  ${DEBUGGING_INTERNAL_HEADERS}
)

list(APPEND SYMBOLIZE_SRC
  "symbolize.cc"
  "symbolize_elf.inc"
  "symbolize_unimplemented.inc"
  "symbolize_win32.inc"
  "internal/demangle.cc"
  ${DEBUGGING_PUBLIC_HEADERS}
  ${DEBUGGING_INTERNAL_HEADERS}
)

list(APPEND FAILURE_SIGNAL_HANDLER_SRC
  "failure_signal_handler.cc"
  ${DEBUGGING_PUBLIC_HEADERS}
)

list(APPEND EXAMINE_STACK_SRC
  "internal/examine_stack.cc"
  ${DEBUGGING_PUBLIC_HEADERS}
  ${DEBUGGING_INTERNAL_HEADERS}
)

absl_library(
  TARGET
    absl_stacktrace
  SOURCES
    ${STACKTRACE_SRC}
  EXPORT_NAME
    stacktrace
)

absl_library(
  TARGET
    absl_symbolize
  SOURCES
    ${SYMBOLIZE_SRC}
  EXPORT_NAME
    symbolize
)

absl_library(
  TARGET
    absl_failure_signal_handler
  SOURCES
    ${FAILURE_SIGNAL_HANDLER_SRC}
  PUBLIC_LIBRARIES
    absl_base absl_synchronization
  EXPORT_NAME
    failure_signal_handler
)

# Internal-only. Projects external to Abseil should not depend
# directly on this library.
absl_library(
  TARGET
    absl_examine_stack
  SOURCES
    ${EXAMINE_STACK_SRC}
  EXPORT_NAME
    examine_stack
)

list(APPEND LEAK_CHECK_SRC
  "leak_check.cc"
)


# leak_check library
absl_library(
  TARGET
    absl_leak_check
  SOURCES
    ${LEAK_CHECK_SRC}
  PUBLIC_LIBRARIES
    absl_base
  EXPORT_NAME
    leak_check
)


# component target
absl_header_library(
  TARGET
    absl_debugging
  PUBLIC_LIBRARIES
    absl_stacktrace absl_leak_check
  EXPORT_NAME
    debugging
)

#
## TESTS
#

list(APPEND DEBUGGING_INTERNAL_TEST_HEADERS
  "internal/stack_consumption.h"
)

list(APPEND STACK_CONSUMPTION_SRC
  "internal/stack_consumption.cc"
  ${DEBUGGING_INTERNAL_TEST_HEADERS}
)

absl_library(
  TARGET
    absl_stack_consumption
  SOURCES
    ${STACK_CONSUMPTION_SRC}
)

absl_test(
  TARGET
    absl_stack_consumption_test
  SOURCES
    ${STACK_CONSUMPTION_SRC}
)

list(APPEND SYMBOLIZE_TEST_SRC "symbolize_test.cc")

absl_test(
  TARGET
    symbolize_test
  SOURCES
    ${SYMBOLIZE_TEST_SRC}
  PUBLIC_LIBRARIES
    absl_symbolize absl_stack_consumption
)

list(APPEND FAILURE_SIGNAL_HANDLER_TEST_SRC "failure_signal_handler_test.cc")

absl_test(
  TARGET
    failure_signal_handler_test
  SOURCES
    ${FAILURE_SIGNAL_HANDLER_TEST_SRC}
  PUBLIC_LIBRARIES
    absl_examine_stack absl_stacktrace absl_symbolize
)

# test leak_check_test
list(APPEND LEAK_CHECK_TEST_SRC "leak_check_test.cc")

absl_test(
  TARGET
    leak_check_test
  SOURCES
    ${LEAK_CHECK_TEST_SRC}
  PUBLIC_LIBRARIES
    absl_leak_check
)
