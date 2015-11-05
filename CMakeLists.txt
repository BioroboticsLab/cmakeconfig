project(cmakeconfig)

#-----------------------------------------------------------------------
# CPM configuration
#-----------------------------------------------------------------------
set(CPM_MODULE_NAME cmakeconfig)

if ((DEFINED CPM_DIR) AND (DEFINED CPM_UNIQUE_ID) AND (DEFINED CPM_TARGET_NAME))
  set(CPM_LIB_TARGET_NAME ${CPM_TARGET_NAME})
  set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CPM_DIR})
  include(CPM)
else()
  set(CPM_DIR "${CMAKE_CURRENT_BINARY_DIR}/cpm-packages" CACHE TYPE STRING)
  find_package(Git)
  if(NOT GIT_FOUND)
    message(FATAL_ERROR "CPM requires Git.")
  endif()
  if (NOT EXISTS ${CPM_DIR}/CPM.cmake)
    message(STATUS "Cloning repo (https://github.com/iauns/cpm)")
    execute_process(
      COMMAND "${GIT_EXECUTABLE}" clone https://github.com/iauns/cpm ${CPM_DIR}
      RESULT_VARIABLE error_code
      OUTPUT_QUIET ERROR_QUIET)
    if(error_code)
      message(FATAL_ERROR "CPM failed to get the hash for HEAD")
    endif()
  endif()
  include(${CPM_DIR}/CPM.cmake)
endif()

CPM_InitModule(${CPM_MODULE_NAME})

macro(get_gcc_version)
    execute_process(COMMAND ${CMAKE_C_COMPILER} -dumpversion
                                    OUTPUT_VARIABLE GCC_VERSION)
endmacro()

macro(biorobotics_release_flags)
    if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang" OR
       "${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
        set(CMAKE_CXX_FLAGS_RELEASE "-O3 -DNDEBUG -DQT_NO_DEBUG")
        # enable C++1y
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++1y -fPIE -fPIC")
        # enable all warnings
            set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra -pedantic -Woverloaded-virtual -Wold-style-cast -Wnon-virtual-dtor -Wsign-promo -Wno-missing-braces")
        # warnings are errors
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Werror -Wno-error=unused-variable -Wno-error=unused-parameter")
    endif()
endmacro()

macro(biorobotics_debug_flags)
    if("${CMAKE_BUILD_TYPE}" STREQUAL "Debug")
        get_gcc_version()
        set(CMAKE_CXX_FLAGS_DEBUG "-Og -ggdb3")
        if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU" AND
                (GCC_VERSION VERSION_GREATER 4.9 OR GCC_VERSION VERSION_EQUAL 4.9))
                set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -fno-omit-frame-pointer")
                set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -fno-optimize-sibling-calls")
                set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -fsanitize=address")
                set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -fsanitize=undefined")
        endif()

        if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
                set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -fno-omit-frame-pointer")
                set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -fno-optimize-sibling-calls")
                set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -fsanitize=address")
                set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -fsanitize=undefined")
                set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -fsanitize=integer")
        endif()
    endif()
endmacro()

macro(biorobotics_handle_msvc)
    if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC")
        # using Visual Studio C++
        # Force to always compile with W3 and treat warnings as errors
        # W4 would be preferable, but causes too many warnings in included files
        if(CMAKE_CXX_FLAGS MATCHES "/W[0-4]")
            string(REGEX REPLACE "/W[0-4]" "/W3" CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
        else()
            set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /W3")
        endif()
            # warnings are errors
            set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /WX")
            # disable warning 4503 on visual studio (boost)
            set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /wd4503")
        add_definitions(-D_CRT_SECURE_NO_WARNINGS)
    endif()
endmacro()

macro(biorobotics_handle_clang)
    if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
            # determine clang version
            EXECUTE_PROCESS( COMMAND ${CMAKE_CXX_COMPILER} --version OUTPUT_VARIABLE clang_full_version_string )
            string (REGEX REPLACE ".*clang version ([0-9]+\\.[0-9]+).*" "\\1" CLANG_VERSION_STRING ${clang_full_version_string})
            if (CLANG_VERSION_STRING VERSION_GREATER 3.5)
                    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-error=inconsistent-missing-override")
            endif()
            # gcc doesn't know about unused-private-field warning
            set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-error=unused-private-field")
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wdocumentation")
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wthread-safety")
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wimplicit-fallthrough")
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wfloat-conversion")
        if(${APPLE})
            set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-inconsistent-missing-override -Wno-deprecated-declarations")
            set(CMAKE_CXX_LINK_FLAGS_DEBUG "-lc++abi")
            set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_CXX_LINK_FLAGS_DEBUG}")
        endif()
    endif()
endmacro()

macro(biorobotics_handle_gcc)
    if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-error=unused-but-set-variable")
        get_gcc_version()
        if (GCC_VERSION VERSION_GREATER 4.9 OR GCC_VERSION VERSION_EQUAL 4.9)
                # implicit conversion warnings
                set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wfloat-conversion")
        endif()
    endif()
endmacro()

macro(get_optional_tag)
    set(extra_args ${ARGN})
    if (NOT "${extra_args}" STREQUAL "")
        set(tag "${extra_args}")
    endif ()
endmacro()

macro(include_pipeline)
    set(tag "master")
    get_optional_tag(${ARGN})
    if(DEFINED PIPELINE_PATH)
        CPM_AddModule("pipeline"
            SOURCE_DIR "${PIPELINE_PATH}")
    else()
        CPM_AddModule("pipeline"
            GIT_REPOSITORY "https://github.com/BioroboticsLab/pipeline.git"
            GIT_TAG "${tag}")
    endif()
endmacro()

macro(include_deeplocalizer_classifier)
    set(tag "master")
    get_optional_tag(${ARGN})
    if(NOT DEFINED DEEPLOCALIZER_CLASSIFIER_PATH)
        CPM_AddModule("deeplocalizer_classifier"
                GIT_REPOSITORY "https://github.com/BioroboticsLab/deeplocalizer_classifier.git"
                GIT_TAG "master"
                EXPORT_MODULE TRUE
        )
    else()
        CPM_AddModule("deeplocalizer_classifier"
                SOURCE_DIR "${DEEPLOCALIZER_CLASSIFIER_PATH}"
                EXPORT_MODULE TRUE
        )
    endif()
endmacro()

macro(include_biotracker_core)
    set(tag "606e6144615a1efda634278efa06497a0bc88681")
    get_optional_tag(${ARGN})
    if(DEFINED BIOTRACKER_CORE_PATH)
        CPM_AddModule("BioTracker"
            SOURCE_DIR "${BIOTRACKER_CORE_PATH}"
        )
    else()
        CPM_AddModule("BioTracker"
            GIT_REPOSITORY "https://github.com/BioroboticsLab/biotracker_core.git"
            # commit from 29.9.2015
            GIT_TAG ${tag}
        )
    endif()
endmacro()

macro(biorobotics_set_compiler_flags)
    biorobotics_config()
endmacro()


macro(biorobotics_config)
    biorobotics_release_flags()
    biorobotics_debug_flags()
    biorobotics_handle_gcc()
    biorobotics_handle_clang()
    biorobotics_handle_msvc()
endmacro()
