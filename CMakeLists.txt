cmake_minimum_required(VERSION 2.6)
project(capstone)

# Compile-time options
# Modify the following options to customize Capstone engine

option(BUILD_DIET "Build diet library" OFF)
option(BUILD_TESTS "Build tests" ON)
option(USE_SYS_DYN_MEM "Use default memory allocation functions" ON)

option(ARM_SUPPORT "ARM support" ON)
option(ARM64_SUPPORT "ARM64 support" ON)
option(MIPS_SUPPORT "MIPS support" ON)
option(PPC_SUPPORT "PowerPC support" ON)
option(SPARC_SUPPORT "Sparc support" ON)
option(SYSZ_SUPPORT "SystemZ support" ON)
option(XCORE_SUPPORT "XCore support" ON)
option(X86_SUPPORT "X86 support" ON)
option(X86_REDUCE "X86 with reduce instruction sets to minimize library" OFF)
# End of compile-time option
# DO NOT modify anything below
include(GenerateExportHeader)

set(VERSION_MAJOR 2)
set(VERSION_MINOR 1)
set(VERSION_PATCH 2)

## sources + definitions
if (USE_SYS_DYN_MEM)
    add_definitions(-DCAPSTONE_USE_SYS_DYN_MEM)
endif ()

set(SOURCES
    cs.c
    MCInst.c
    MCInstrDesc.c
    MCRegisterInfo.c
    SStream.c
    utils.c
    )

set(TEST_SOURCES test.c test_detail.c test_skipdata.c)

if (ARM_SUPPORT)
    add_definitions(-DCAPSTONE_HAS_ARM)
    set(SOURCES
        ${SOURCES}
        arch/ARM/ARMDisassembler.c
        arch/ARM/ARMInstPrinter.c
        arch/ARM/ARMMapping.c
        arch/ARM/ARMModule.c
        )
    set(TEST_SOURCES ${TEST_SOURCES} test_arm.c)
endif ()

if (ARM64_SUPPORT)
    add_definitions(-DCAPSTONE_HAS_ARM64)
    set(SOURCES
        ${SOURCES}
        arch/AArch64/AArch64BaseInfo.c
        arch/AArch64/AArch64Disassembler.c
        arch/AArch64/AArch64InstPrinter.c
        arch/AArch64/AArch64Mapping.c
        arch/AArch64/AArch64Module.c
        )
    set(TEST_SOURCES ${TEST_SOURCES} test_arm64.c)
endif ()

if (MIPS_SUPPORT)
    add_definitions(-DCAPSTONE_HAS_MIPS)
    set(SOURCES
        ${SOURCES}
        arch/Mips/MipsDisassembler.c
        arch/Mips/MipsInstPrinter.c
        arch/Mips/MipsMapping.c
        arch/Mips/MipsModule.c
        )
    set(TEST_SOURCES ${TEST_SOURCES} test_mips.c)
endif ()

if (PPC_SUPPORT)
    add_definitions(-DCAPSTONE_HAS_POWERPC)
    set(SOURCES
        ${SOURCES}
        arch/PowerPC/PPCDisassembler.c
        arch/PowerPC/PPCInstPrinter.c
        arch/PowerPC/PPCMapping.c
        arch/PowerPC/PPCModule.c
        )
    set(TEST_SOURCES ${TEST_SOURCES} test_ppc.c)
endif ()

if (X86_SUPPORT)
    add_definitions(-DCAPSTONE_HAS_X86)
if (BUILD_DIET)
    set(SOURCES
        ${SOURCES}
        arch/X86/X86Disassembler.c
        arch/X86/X86DisassemblerDecoder.c
        arch/X86/X86IntelInstPrinter.c
        arch/X86/X86Mapping.c
        arch/X86/X86Module.c
        )
else ()
    set(SOURCES
        ${SOURCES}
        arch/X86/X86ATTInstPrinter.c
        arch/X86/X86Disassembler.c
        arch/X86/X86DisassemblerDecoder.c
        arch/X86/X86IntelInstPrinter.c
        arch/X86/X86Mapping.c
        arch/X86/X86Module.c
        )
endif ()
    set(TEST_SOURCES ${TEST_SOURCES} test_x86.c)
endif ()

if (SPARC_SUPPORT)
    add_definitions(-DCAPSTONE_HAS_SPARC)
    set(SOURCES
        ${SOURCES}
	arch/Sparc/SparcDisassembler.c
	arch/Sparc/SparcInstPrinter.c
	arch/Sparc/SparcMapping.c
	arch/Sparc/SparcModule.c
        )
    set(TEST_SOURCES ${TEST_SOURCES} test_sparc.c)
endif ()

if (SYSZ_SUPPORT)
    add_definitions(-DCAPSTONE_HAS_SYSZ)
    set(SOURCES
        ${SOURCES}
	arch/SystemZ/SystemZDisassembler.c
	arch/SystemZ/SystemZInstPrinter.c
	arch/SystemZ/SystemZMapping.c
	arch/SystemZ/SystemZModule.c
	arch/SystemZ/SystemZMCTargetDesc.c
        )
    set(TEST_SOURCES ${TEST_SOURCES} test_systemz.c)
endif ()

if (XCORE_SUPPORT)
    add_definitions(-DCAPSTONE_HAS_XCORE)
    set(SOURCES
        ${SOURCES}
	arch/XCore/XCoreDisassembler.c
	arch/XCore/XCoreInstPrinter.c
	arch/XCore/XCoreMapping.c
	arch/XCore/XCoreModule.c
        )
    set(TEST_SOURCES ${TEST_SOURCES} test_xcore.c)
endif ()

include_directories("${PROJECT_SOURCE_DIR}/include")

if (BUILD_DIET)
    add_definitions(-DCAPSTONE_DIET)
endif ()

if (X86_REDUCE)
    add_definitions(-DCAPSTONE_X86_REDUCE)
endif ()

## targets
add_library(libcapstone_static STATIC ${SOURCES})
add_library(libcapstone SHARED ${SOURCES})
generate_export_header(libcapstone
                       BASE_NAME capstone
                       EXPORT_FILE_NAME ${PROJECT_SOURCE_DIR}/include/capstone-export.h)

#include_directories("${PROJECT_SOURCE_DIR}/include")

if (BUILD_TESTS)
    foreach (TSRC ${TEST_SOURCES})
        STRING(REGEX REPLACE ".c$" "" TBIN ${TSRC})
        add_executable(${TBIN} "tests/${TSRC}")
        target_link_libraries(${TBIN} libcapstone_static)
    endforeach ()
endif ()

## properties
set_property(GLOBAL PROPERTY VERSION ${VERSION_MAJOR}.${VERSION_MINOR}.${VERSION_PATCH})
set_property(GLOBAL PROPERTY SOVERSION ${VERSION_MAJOR})
set_property(TARGET libcapstone_static PROPERTY COMPILE_FLAGS -DCAPSTONE_STATIC_DEFINE)

## install
set(INCLUDES arm64.h  arm.h  capstone.h  mips.h  ppc.h  x86.h sparc.h systemz.h xcore.h)
foreach (INC ${INCLUDES})
    install(FILES "include/${INC}" DESTINATION include/capstone)
endforeach ()

install(TARGETS libcapstone
        RUNTIME DESTINATION bin
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib)
