cmake_minimum_required(VERSION 3.28)

project(cuda)

set(CUDA_VERSION "13.3.1")
set(CUDA_DRIVER_VERSION "610.43.02")

set(CUDA_INSTALLER
    "cuda_${CUDA_VERSION}_${CUDA_DRIVER_VERSION}_linux.run"
)

set(CUDA_URL
    "https://developer.download.nvidia.com/compute/cuda/${CUDA_VERSION}/local_installers/${CUDA_INSTALLER}"
)

set(CUDA_MD5_URL
    "https://developer.download.nvidia.com/compute/cuda/${CUDA_VERSION}/docs/sidebar/md5sum.txt"
)


include(ExternalProject)

# get MD5 checksum for the installer
file(DOWNLOAD ${CUDA_MD5_URL} ${CMAKE_BINARY_DIR}/md5sum.txt)
file(STRINGS ${CMAKE_BINARY_DIR}/md5sum.txt CUDA_MD5_LINES)
foreach(line IN LISTS CUDA_MD5_LINES)
    # message(STATUS "Checking line: ${line}")
    string(REGEX MATCH "^([^ \t]+)[ \t]+${CUDA_INSTALLER}" MATCHED "${line}")
    if(MATCHED)
        set(CUDA_INSTALLER_MD5 "${CMAKE_MATCH_1}")
        break()
    endif()
endforeach()
if(NOT CUDA_INSTALLER_MD5)
    message(FATAL_ERROR "MD5 entry for '${CUDA_INSTALLER}' not found in ${CUDA_MD5_URL}")
endif()
message(STATUS "CUDA installer MD5: ${CUDA_INSTALLER_MD5}")

# download CUDA installer
ExternalProject_Add(cuda_installer
    URL                 ${CUDA_URL}
    URL_MD5             ${CUDA_INSTALLER_MD5}
    DOWNLOAD_DIR        ${CMAKE_BINARY_DIR}
    DOWNLOAD_NO_EXTRACT True
    # DOWNLOAD_EXTRACT_TIMESTAMP False
    CONFIGURE_COMMAND   ""
    # BUILD_COMMAND       ""
    # BUILD_COMMAND       sh <DOWNLOADED_FILE> --silent --no-man-page --toolkit --toolkitpath=${CMAKE_BINARY_DIR}/cuda_toolkit/
    # BUILD_COMMAND       sh -c "DISPLAY= sh <DOWNLOADED_FILE> --extract=${CMAKE_BINARY_DIR}/cuda_toolkit/"
    BUILD_COMMAND       ${CMAKE_COMMAND} -E env --unset=DISPLAY sh <DOWNLOADED_FILE> --extract=${CMAKE_BINARY_DIR}/cuda_toolkit/
    INSTALL_COMMAND     ""
    # UPDATE_COMMAND ""
)

set(CUDA_TOOLKIT_SUBFOLDERS
    # bin
    cccl
    cuda_crt
    cuda_ctadvisor
    cuda_cudart
    cuda_culibos
    cuda_cuobjdump
    cuda_cupti
    cuda_cuxxfilt
    # cuda_documentation
    cuda_gdb
    cuda_nvcc
    cuda_nvdisasm
    cuda_nvml_dev
    cuda_nvprune
    cuda_nvrtc
    cuda_nvtx
    cuda_opencl
    cuda_profiler_api
    cuda_sandbox_dev
    cuda_sanitizer_api
    cuda_tileiras
    # integration
    libcublas
    libcufft
    # libcufile
    # libcuobjclient
    libcurand
    # libcusolver
    libcusparse
    libnpp
    libnvfatbin
    libnvjitlink
    libnvjpeg
    libnvptxcompiler
    libnvvm
    # nsight_compute
    # nsight_systems
    # nvidia_fs
)

foreach(subfolder IN LISTS CUDA_TOOLKIT_SUBFOLDERS)
    install(
        DIRECTORY   "${CMAKE_BINARY_DIR}/cuda_toolkit/${subfolder}/"
        DESTINATION "${CMAKE_INSTALL_PREFIX}"
    )
endforeach()
