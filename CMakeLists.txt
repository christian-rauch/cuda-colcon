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
    BUILD_COMMAND       sh <DOWNLOADED_FILE> --silent --no-man-page --toolkit --toolkitpath=${CMAKE_BINARY_DIR}/cuda_toolkit/
    # INSTALL_COMMAND     sh <DOWNLOADED_FILE> --no-man-page --toolkit --toolkitpath=${CMAKE_INSTALL_PREFIX}
    INSTALL_COMMAND     ""
    # UPDATE_COMMAND ""
)

# noop install instruction to silence warning
# install(CODE "")

install(DIRECTORY ${CMAKE_BINARY_DIR}/cuda_toolkit/
    DESTINATION ${CMAKE_INSTALL_PREFIX}
)
