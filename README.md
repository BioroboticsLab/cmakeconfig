# BioroboticsLab cmakeconfig

The main aim of this project is to have the same compiler flags on the different
[BioroboticsLab](https://github.com/BioroboticsLab) projects. It also provides
convenient functions to include [BioroboticsLab](https://github.com/BioroboticsLab)
projects.

# Include cmakeconfig in your project

Use this config to include cmakeconfig into your project.
`cmakeconfig` will download [cpm](iauns/cpm), so there is no need to also add
the cpm boilercode.

```cmake
set(CPM_MODULE_NAME <YourModule>)
set(CPM_LIB_TARGET_NAME ${CPM_MODULE_NAME})
if ((DEFINED BIOROBOTICS_CMAKE_CONFIG_DIR))
    include(BIOROBOTICS_CMAKE_CONFIG_DIR)
else()
    set(BIOROBOTICS_CMAKE_CONFIG_DIR "${CMAKE_CURRENT_BINARY_DIR}/cmakeconfig" CACHE TYPE STRING)
    find_package(Git)
    if(NOT GIT_FOUND)
      message(FATAL_ERROR "CPM requires Git.")
    endif()
    if (NOT EXISTS ${BIOROBOTICS_CMAKE_CONFIG_DIR}/CMakeLists.txt)
      message(STATUS "Cloning repo (https://github.com/BioroboticsLab/cmakeconfig.git)")
      execute_process(
        COMMAND "${GIT_EXECUTABLE}" clone https://github.com/BioroboticsLab/cmakeconfig.git
        ${BIOROBOTICS_CMAKE_CONFIG_DIR}
        RESULT_VARIABLE error_code
        OUTPUT_QUIET ERROR_QUIET)
      if(error_code)
          message(FATAL_ERROR "CMAKECONFIG failed to get the hash for HEAD")
      endif()
    endif()
    include(${BIOROBOTICS_CMAKE_CONFIG_DIR}/CMakeLists.txt)
endif()

CPM_AddModule(...)
...
CPM_InitModule(${CPM_MODULE_NAME})

biorobotics_set_compiler_flags()
```

# Include other BioroboticsLab Repositories

Every include macro has an optional **git_tag** and you can use a local copy
of the dependency by setting the corresponding PATH variable.
For example for the pipeline:

```$ cmake -DPIPELINE_PATH=~/dev/pipeline ```

This uses your local copy of the pipeline at `~/dev/pipeline`.


* **`include_pipeline()`**: Includes the [pipeline](https://github.com/BioroboticsLab/pipeline).
    Set local directory with `PIPELINE_PATH`

* **`include_biotracker_core()`**: Includes the [biotracker_core](https://github.com/BioroboticsLab/biotracker_core). Set local directory with `BIOTRACKER_CORE_PATH`

* **`include_deeplocalizer_classifier(git_tag)`**: Includes the [deeplocalizer_classifier](https://github.com/BioroboticsLab/deeplocalizer_classifier). Set local directory with `DEEPLOCALIZER_CLASSIFIER_PATH` variable.

