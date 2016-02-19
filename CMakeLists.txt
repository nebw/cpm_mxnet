#-----------------------------------------------------------------------
# CPM configuration
#-----------------------------------------------------------------------
set(CPM_MODULE_NAME cpm_mxnet)

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
    execute_process(
      COMMAND "${GIT_EXECUTABLE}" clone https://github.com/iauns/cpm ${CPM_DIR}
      RESULT_VARIABLE error_code
      OUTPUT_VARIABLE head_sha)
    if(error_code)
      message(FATAL_ERROR "CPM failed to get the hash for HEAD")
    endif()
  endif()
  include(${CPM_DIR}/CPM.cmake)
endif()

# All externals *must* define this.
CPM_ForceOnlyOneModuleVersion()

CPM_InitModule(${CPM_MODULE_NAME})

#------------------------------------------------------------------------------
# MXNet
#------------------------------------------------------------------------------

set(USE_OPENCV ON)
set(USE_CUDNN OFF)
set(USE_CUDA OFF)

if(NOT DEFINED MXNET_COMMIT)
    set(MXNET_COMMIT "a1b07486f70486659ff3f78f673bc21024549238")
endif()

if(NOT DEFINED MXNET_GIT_REPO)
    set(MXNET_GIT_REPO "https://github.com/dmlc/mxnet.git")
endif()

set(REPO_SOURCE_DIR "${CMAKE_CURRENT_BINARY_DIR}/3rdParty/mxnet")

CPM_EnsureRepoIsCurrent(
  TARGET_DIR "${REPO_SOURCE_DIR}"
  GIT_REPOSITORY "${MXNET_GIT_REPO}"
  GIT_TAG "${MXNET_COMMIT}"
  USE_CACHING TRUE
)
CPM_ExportAdditionalIncludeDir("${REPO_SOURCE_DIR}/include")
CPM_ExportAdditionalLibraryTarget(mxnet)

add_subdirectory("${REPO_SOURCE_DIR}" "${CMAKE_CURRENT_BINARY_DIR}/3rdParty/build/mxnet}")
