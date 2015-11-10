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
#------------------------------------------------------------------------------
# Required CPM Setup - no need to modify - See: https://github.com/iauns/cpm
#------------------------------------------------------------------------------
set(CPM_DIR "${CMAKE_CURRENT_BINARY_DIR}/cpm_packages" CACHE TYPE STRING)
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


CPM_AddModule("cmakeconfig"
    GIT_REPOSITORY "https://github.com/BioroboticsLab/cmakeconfig.git"
    GIT_TAG "master")
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

