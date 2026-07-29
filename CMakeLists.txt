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
    message(STATUS "Checking line: ${line}")

    # if(line MATCHES "^([0-9a-fA-F]{32})[ \t]+${CUDA_INSTALLER}$")
    #     set(CUDA_INSTALLER_MD5 "${CMAKE_MATCH_1}")
    #     break()
    # endif()

    string(REGEX MATCH "^([^ \t]+)[ \t]+${CUDA_INSTALLER}" MATCHED "${line}")
    if(MATCHED)
        set(CUDA_INSTALLER_MD5 "${CMAKE_MATCH_1}")
        message(STATUS "Checksum: ${CUDA_INSTALLER_MD5}")
        break()
    endif()
endforeach()
if(NOT CUDA_INSTALLER_MD5)
    message(FATAL_ERROR "MD5 entry for '${CUDA_INSTALLER}' not found in ${CUDA_MD5_URL}")
endif()
message(STATUS "CUDA installer MD5: ${CUDA_INSTALLER_MD5}")

# download CUDA installer
ExternalProject_Add(
    cuda_installer
    URL ${CUDA_URL}
    URL_MD5 ${CUDA_INSTALLER_MD5}
    # DOWNLOAD_DIR ${CUDA_DOWNLOAD_DIR}
    DOWNLOAD_NO_EXTRACT True
    # DOWNLOAD_EXTRACT_TIMESTAMP False
    CONFIGURE_COMMAND ""
    BUILD_COMMAND ""
    INSTALL_COMMAND ""
    UPDATE_COMMAND ""
)


# # Install CUDA toolkit
# ExternalProject_Add(
#     cuda_install
#     DEPENDS cuda_download

#     CONFIGURE_COMMAND ""
#     BUILD_COMMAND ""

#     INSTALL_COMMAND
#         sudo bash ${CUDA_DOWNLOAD_DIR}/${CUDA_INSTALLER}
#         --silent
#         --toolkit
#         --toolkitpath=${CUDA_PREFIX}

#     UPDATE_COMMAND ""
# )


# # Create /usr/local/cuda symlink
# add_custom_target(cuda ALL
#     DEPENDS cuda_install

#     COMMAND sudo ln -sfn
#             ${CUDA_PREFIX}
#             /usr/local/cuda
# )
