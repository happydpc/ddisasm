# ===== generated souffle code =====
# On windows, wsl is used to call souffle.  There is no good way to 'find'
# souffle in wsl, so we skip this step entirely and will hard-code the call.
if(NOT WIN32)
  find_program(
    SOUFFLE souffle
    HINTS $ENV{PATH}
    DOC "souffle must be in your PATH to build disassembler.")
  if(NOT SOUFFLE)
    message(FATAL_ERROR "souffle was not found in your PATH. Unable to build.")
  endif()
else()
  set(SOUFFLE wsl souffle)
endif()

set(DATALOG_SOURCES
    datalog/bitmasks.dl
    datalog/code_inference.dl
    datalog/code_inference_postprocess.dl
    datalog/cfg.dl
    datalog/data_access_analysis.dl
    datalog/empty_range.dl
    datalog/elf_binaries.dl
    datalog/exceptions.dl
    datalog/basic_function_inference.dl
    datalog/main.dl
    datalog/pointer_reatribution.dl
    datalog/printable_chars.dl
    datalog/relative_jump_tables.dl
    datalog/symbolization.dl
    datalog/use_def_analysis.dl
    datalog/value_analysis.dl
    datalog/debug_stats.dl
    datalog/self_diagnose.dl
    datalog/x64/arch_x64.dl
    datalog/x64/float_operations.dl
    datalog/x64/jump_operations.dl
    datalog/x64/registers.dl)

if(WIN32)
  set(SOUFFLE_DISASM_MAIN
      "$$(wslpath ${CMAKE_CURRENT_SOURCE_DIR}/datalog/main.dl)")
else()
  set(SOUFFLE_DISASM_MAIN ${CMAKE_CURRENT_SOURCE_DIR}/datalog/main.dl)
endif()

set(GENERATED_CPP "${CMAKE_BINARY_DIR}/src/souffle_disasm.cpp")
add_custom_command(
  OUTPUT ${GENERATED_CPP}
         # Souffle includes the path of the output file in the generated program
         # name. Change directory and use a relative path so the name does not
         # depend on build location.
  WORKING_DIRECTORY "${CMAKE_BINARY_DIR}/src/"
  COMMAND ${SOUFFLE} ${SOUFFLE_DISASM_MAIN} -g souffle_disasm.cpp -jauto
  DEPENDS ${DATALOG_SOURCES})

# determine what flags to use to specify -fopenmp.
if(${CMAKE_CXX_COMPILER_ID} STREQUAL GNU)
  set(OPENMP_FLAGS -fopenmp)
elseif(${CMAKE_CXX_COMPILER_ID} STREQUAL Clang)
  set(OPENMP_FLAGS -fopenmp=libgomp)
endif()

# Common settings for MSVC
function(set_common_msvc_options TARGET_NAME)
  target_compile_options(${TARGET_NAME} PRIVATE "-WX")
  target_compile_options(${TARGET_NAME} PRIVATE "-permissive-")
  target_compile_options(${TARGET_NAME} PRIVATE "-EHsc")
  target_compile_options(${TARGET_NAME} PRIVATE "-bigobj")

  target_compile_options(${TARGET_NAME} PRIVATE $<$<CONFIG:Debug>:-D_DEBUG>)
  target_compile_options(${TARGET_NAME} PRIVATE $<$<CONFIG:Debug>:-MDd>)
  target_compile_options(${TARGET_NAME} PRIVATE $<$<CONFIG:Debug>:-Ob0>
  )# Disables inline expansion
  target_compile_options(
    ${TARGET_NAME} PRIVATE $<$<CONFIG:Debug>:-Od>) # Disables optimization,
                                                   # speeding compilation and
                                                   # simplifying debugging. http
                                                   # s://msdn.microsoft.com/en-
                                                   # us/library/k1ack8f1.aspx
  target_compile_options(${TARGET_NAME} PRIVATE $<$<CONFIG:Debug>:-RTC1>
  )# Enables run-time error checking.
  target_compile_options(${TARGET_NAME} PRIVATE $<$<CONFIG:Debug>:-W4>
  )# Sets warning level.
  target_compile_options(${TARGET_NAME} PRIVATE $<$<CONFIG:Debug>:-Zi>
  )# Generates complete debugging information.

  target_compile_options(${TARGET_NAME}
                         PRIVATE $<$<CONFIG:RelWithDebInfo>:-D_NDEBUG>)
  target_compile_options(${TARGET_NAME}
                         PRIVATE $<$<CONFIG:RelWithDebInfo>:-DNDEBUG>)
  target_compile_options(${TARGET_NAME} PRIVATE $<$<CONFIG:RelWithDebInfo>:-MD>)
  target_compile_options(${TARGET_NAME} PRIVATE $<$<CONFIG:RelWithDebInfo>:-O2>
  )# Creates fast code.
  target_compile_options(
    ${TARGET_NAME} PRIVATE $<$<CONFIG:RelWithDebInfo>:-Ob2>) # The default
                                                             # value. Allows
                                                             # expansion of
                                                             # functions marked
                                                             # as inline,
                                                             # __inline, or
                                                             # __forceinline,
                                                             # and any other
                                                             # function that the
                                                             # compiler chooses.
  target_compile_options(${TARGET_NAME} PRIVATE $<$<CONFIG:RelWithDebInfo>:-Oi>
  )# Generates intrinsic functions.
  target_compile_options(${TARGET_NAME} PRIVATE $<$<CONFIG:RelWithDebInfo>:-Ot>
  )# Favors fast code.
  target_compile_options(${TARGET_NAME} PRIVATE $<$<CONFIG:RelWithDebInfo>:-W4>
  )# Sets warning level.
  target_compile_options(${TARGET_NAME} PRIVATE $<$<CONFIG:RelWithDebInfo>:-Zi>
  )# Generates complete debugging information.

  target_compile_options(${TARGET_NAME} PRIVATE $<$<CONFIG:Release>:-D_NDEBUG>)
  target_compile_options(${TARGET_NAME} PRIVATE $<$<CONFIG:Release>:-DNDEBUG>)
  target_compile_options(${TARGET_NAME} PRIVATE $<$<CONFIG:Release>:-MD>)
  target_compile_options(${TARGET_NAME} PRIVATE $<$<CONFIG:Release>:-O2>
  )# Creates fast code.
  target_compile_options(
    ${TARGET_NAME} PRIVATE $<$<CONFIG:Release>:-Ob2>) # The default value.
                                                      # Allows expansion of
                                                      # functions marked as
                                                      # inline, __inline, or
                                                      # __forceinline, and any
                                                      # other function that the
                                                      # compiler chooses.
  target_compile_options(${TARGET_NAME} PRIVATE $<$<CONFIG:Release>:-Oi>
  )# Generates intrinsic functions.
  target_compile_options(${TARGET_NAME} PRIVATE $<$<CONFIG:Release>:-Ot>
  )# Favors fast code.
  target_compile_options(${TARGET_NAME} PRIVATE $<$<CONFIG:Release>:-W4>
  )# Sets warning level.
endfunction()

# Disabled warnings for souffle projects on MSVC
function(set_souffle_msvc_options TARGET_NAME)
  target_compile_definitions(${TARGET_NAME} PRIVATE _CRT_SECURE_NO_WARNINGS)
  target_compile_definitions(${TARGET_NAME} PRIVATE _CRT_NONSTDC_NO_WARNINGS)

  # From cpp generated by souffle:
  target_compile_options(
    ${TARGET_NAME} PRIVATE -wd4146) # unary minus operator applied to unsigned
                                    # type, result still unsigned
  target_compile_options(
    ${TARGET_NAME} PRIVATE -wd4189) # 'identifier' : local variable is
                                    # initialized but not referenced

  # From souffle headers
  target_compile_options(
    ${TARGET_NAME} PRIVATE -wd4267) # conversion from 'type1' to 'type2',
                                    # possible loss of data
  target_compile_options(
    ${TARGET_NAME} PRIVATE -wd4244) # 'conversion' conversion from 'type1' to
                                    # 'type2', possible loss of data
  target_compile_options(
    ${TARGET_NAME} PRIVATE -wd4456) # declaration of 'decl' hides previous local
                                    # declaration
endfunction()

# ====== datalog_utils ======

add_library(datalog_utils STATIC DatalogUtils.cpp DlOperandTable.cpp)

target_link_libraries(datalog_utils gtirb ${CAPSTONE})

target_compile_definitions(datalog_utils PRIVATE __EMBEDDED_SOUFFLE__)
target_compile_definitions(datalog_utils PRIVATE RAM_DOMAIN_SIZE=64)
target_compile_options(datalog_utils PRIVATE ${OPENMP_FLAGS})

if(${CMAKE_CXX_COMPILER_ID} STREQUAL MSVC)
  target_compile_definitions(datalog_utils PRIVATE _CRT_SECURE_NO_WARNINGS)
  target_compile_definitions(datalog_utils PRIVATE _CRT_NONSTDC_NO_WARNINGS)

  set_common_msvc_options(datalog_utils)
else()
  target_compile_options(datalog_utils PRIVATE -O3)
endif()
# ====== passes ============

add_subdirectory(passes)

# ===== souffle_disasm =====
# First build a static library of all the non-generated code.. This is just a
# hack to get CMake to use different compile flags (because the generated
# souffle code won't build with -Wall -Werror).
add_library(
  disasm_main STATIC
  disasm_main.cpp DlDecoder.cpp ExceptionDecoder.cpp
  GtirbModuleDisassembler.cpp GtirbZeroBuilder.cpp LIEFBinaryReader.cpp)

target_compile_definitions(disasm_main PRIVATE __EMBEDDED_SOUFFLE__)
target_compile_definitions(disasm_main PRIVATE RAM_DOMAIN_SIZE=64)
target_compile_options(disasm_main PRIVATE ${OPENMP_FLAGS})
if(${CMAKE_CXX_COMPILER_ID} STREQUAL MSVC)
  target_compile_definitions(disasm_main PRIVATE _CRT_SECURE_NO_WARNINGS)
  target_compile_definitions(disasm_main PRIVATE _CRT_NONSTDC_NO_WARNINGS)

  # Warnings from LIEF
  target_compile_options(
    disasm_main PRIVATE -wd4245) # conversion from 'type1' to 'type2',
                                 # signed/unsigned mismatch
  target_compile_options(
    disasm_main PRIVATE -wd4146) # unary minus operator applied to unsigned
                                 # type, result still unsigned
  target_compile_options(
    disasm_main PRIVATE -wd4267) # conversion from 'size_t' to 'type', possible
                                 # loss of data
  target_compile_options(
    disasm_main PRIVATE -wd4359) # actual alignment (8) is greater than the
                                 # value specified in __declspec(align())

  set_common_msvc_options(disasm_main)
else()
  target_compile_options(disasm_main PRIVATE -O3)
  target_compile_options(disasm_main PRIVATE -Wall)
  target_compile_options(disasm_main PRIVATE -Wextra -Wpointer-arith)
  target_compile_options(disasm_main PRIVATE -Werror)
endif()

if(${GTIRB_USE_SYSTEM_BOOST} MATCHES "OFF")
  add_dependencies(disasm_main Boost)
endif()

# ============ Detect how to use <experimental/filesystem>
# ============================
include(CheckCXXSourceCompiles)

# Detect how to use <experimental/filesystem> for non-msvc compilers.  On the
# latest version of MSVC, the experimental filesystem library is deprecated.
set(USE_STD_FILESYSTEM_LIB_DEFAULT OFF)
if(${CMAKE_CXX_COMPILER_ID} STREQUAL MSVC)
  set(USE_STD_FILESYSTEM_LIB_DEFAULT ON)
endif()

option(
  USE_STD_FILESYSTEM_LIB
  "Use the standard filesystem library instead of the experimental filesystem library"
  ${USE_STD_FILESYSTEM_LIB_DEFAULT})

if(USE_STD_FILESYSTEM_LIB)
  add_definitions(-DUSE_STD_FILESYSTEM_LIB)
else()
  set(EXPERIMENTAL_LIB
      ""
      CACHE
        STRING
        "The library to link with in order to use <experimental/filesystem>.")

  if(NOT EXPERIMENTAL_LIB)
    set(EXFS_CODE
        "
    #include <experimental/filesystem>
    int main() {
      std::experimental::filesystem::path p{\"an/example/path\"};
      return std::experimental::filesystem::exists(p);
    }
    ")

    check_cxx_source_compiles("${EXFS_CODE}" EXFS_LINK_NONE)

    if(NOT EXFS_LINK_NONE)
      foreach(EXFS_LIB stdc++fs c++experimental c++fs)
        set(OLD_CMAKE_REQUIRED_LIBRARIES ${CMAKE_REQUIRED_LIBRARIES})
        list(APPEND CMAKE_REQUIRED_LIBRARIES ${EXFS_LIB})
        check_cxx_source_compiles("${EXFS_CODE}" EXFS_LINK_${EXFS_LIB}_OK)
        set(CMAKE_REQUIRED_LIBRARIES ${OLD_CMAKE_REQUIRED_LIBRARIES})

        if(EXFS_LINK_${EXFS_LIB}_OK)
          if(EXPERIMENTAL_LIB)
            message(
              FATAL_ERROR
                "multiple candidates for library to link for <experimental/filesystem>: ${EXPERIMENTAL_LIB} and ${EXFS_LIB}. Try giving the correct one explicitly with -DEXPERIMENTAL_LIB."
            )
          endif()
          set(EXPERIMENTAL_LIB ${EXFS_LIB})
        endif()
      endforeach()

      if(NOT EXPERIMENTAL_LIB)
        message(
          FATAL_ERROR
            "could not find library to link for <experimental/filesystem>. Try giving it explicitly with -DEXPERIMENTAL_LIB."
        )
      endif()
    endif()
  endif()
endif()

target_include_directories(disasm_main PUBLIC ${LIEF_INCLUDE_DIRS})

target_link_libraries(
  disasm_main
  gtirb
  gtirb_pprinter
  ehp
  datalog_utils
  ${CAPSTONE}
  ${Boost_LIBRARIES}
  ${EXPERIMENTAL_LIB}
  ${LIBCPP_ABI}
  ${LIEF_LIBRARIES})

# Now combine the static library and generated code into an executable.
add_executable(ddisasm ${GENERATED_CPP})

if(${CMAKE_CXX_COMPILER_ID} STREQUAL MSVC)
  target_link_libraries(ddisasm disasm_main scc_pass no_return_pass
                        function_inference_pass)
  target_link_options(
    ddisasm PRIVATE /WHOLEARCHIVE:no_return_pass$<$<CONFIG:Debug>:d>
    /WHOLEARCHIVE:function_inference_pass$<$<CONFIG:Debug>:d>)
else()
  target_link_libraries(
    ddisasm
    disasm_main
    scc_pass
    -Wl,--whole-archive
    no_return_pass
    function_inference_pass
    -Wl,--no-whole-archive)
endif()

target_compile_definitions(ddisasm PRIVATE __EMBEDDED_SOUFFLE__)
target_compile_definitions(ddisasm PRIVATE RAM_DOMAIN_SIZE=64)
target_compile_options(ddisasm PRIVATE ${OPENMP_FLAGS})

if(${CMAKE_CXX_COMPILER_ID} STREQUAL MSVC)
  target_link_options(ddisasm PRIVATE -NODEFAULTLIB:LIBCMTD)

  set_common_msvc_options(ddisasm)

  set_souffle_msvc_options(ddisasm)
else()
  target_compile_options(ddisasm PRIVATE -O3)
  target_compile_options(ddisasm PRIVATE -Wno-parentheses-equality)
endif()

if(${CMAKE_CXX_COMPILER_ID} STREQUAL GNU)
  target_link_libraries(ddisasm gomp)
endif()

if(DDISASM_ENABLE_TESTS)
  add_subdirectory(tests)
endif()

install(TARGETS ddisasm DESTINATION bin)

if(NOT BUILD_SHARED_LIBS)
  target_link_libraries(ddisasm -static-libstdc++)
endif()
