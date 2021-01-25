file(MAKE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/xnu_header_install)
externalproject_add(xnu_headers.extproj
    PREFIX ${CMAKE_CURRENT_BINARY_DIR}/xnu_header_build
    SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}
    CONFIGURE_COMMAND
        cmake
            -D XNU_SRC=${CMAKE_CURRENT_SOURCE_DIR}
            -D CODESIGN_ALLOCATE_PATH=$<TARGET_FILE:darwin_codesign_allocate>
            -D CTFCONVERT_PATH=$<TARGET_FILE:ctfconvert.host>
            -D CTFMERGE_PATH=$<TARGET_FILE:ctfmerge.host>
            -D AVAILABILITY_PL_PATH=$<TARGET_FILE:availability.pl>
            -P ${CMAKE_CURRENT_SOURCE_DIR}/preprocess_files.cmake
    BUILD_COMMAND
        make -C <SOURCE_DIR> installhdrs ARCH_CONFIGS=X86_64 BUILD_LTO=0
            BUILD_WERROR=0 DO_CTFMERGE=0 KERNEL_CONFIGS=RELEASE
            SRCROOT=${CMAKE_CURRENT_SOURCE_DIR} OBJROOT=<BINARY_DIR>
            DSTROOT=${CMAKE_CURRENT_BINARY_DIR}/xnu_header_install
    INSTALL_COMMAND true # installed during build
    UPDATE_COMMAND "" # prevent constant rebuilds
    USES_TERMINAL_CONFIGURE TRUE
    USES_TERMINAL_BUILD TRUE
)

add_library(xnu_headers INTERFACE)
add_dependencies(xnu_headers xnu_headers.extproj)
target_include_directories(xnu_headers INTERFACE
    ${CMAKE_CURRENT_BINARY_DIR}/xnu_header_install/usr/include
)
add_library(xnu_private_headers INTERFACE)
add_dependencies(xnu_private_headers xnu_headers.extproj)
target_include_directories(xnu_private_headers INTERFACE
    ${CMAKE_CURRENT_BINARY_DIR}/xnu_header_install/usr/local/include
)
add_library(xnu_kernel_headers INTERFACE)
add_dependencies(xnu_kernel_headers xnu_headers.extproj)
target_include_directories(xnu_kernel_headers INTERFACE
    ${CMAKE_CURRENT_BINARY_DIR}/xnu_header_install/System/Library/Frameworks/Kernel.framework/Versions/A/Headers
)
add_library(xnu_kernel_private_headers INTERFACE)
add_dependencies(xnu_kernel_private_headers xnu_headers.extproj)
target_include_directories(xnu_kernel_private_headers INTERFACE
    ${CMAKE_CURRENT_BINARY_DIR}/xnu_header_install/System/Library/Frameworks/Kernel.framework/Versions/A/PrivateHeaders
)



externalproject_add(xnu
    PREFIX ${CMAKE_CURRENT_BINARY_DIR}/xnu_build
    SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}
    CONFIGURE_COMMAND
        cmake
            -D XNU_SRC=${CMAKE_CURRENT_SOURCE_DIR}
            -D CODESIGN_ALLOCATE_PATH=$<TARGET_FILE:darwin_codesign_allocate>
            -D CTFCONVERT_PATH=$<TARGET_FILE:ctfconvert.host>
            -D CTFMERGE_PATH=$<TARGET_FILE:ctfmerge.host>
            -D AVAILABILITY_PL_PATH=$<TARGET_FILE:availability.pl>
            -D LIBFIREHOSE_HEADER_PATH=${PUREDARWIN_SOURCE_DIR}/projects/libdispatch
            -D LIBFIREHOSE_LIB_PATH=$<TARGET_FILE_DIR:libfirehose_kernel>
            -P ${CMAKE_CURRENT_SOURCE_DIR}/preprocess_files.cmake
    BUILD_COMMAND
        make -C <SOURCE_DIR> install ARCH_CONFIGS=X86_64 BUILD_LTO=0
            BUILD_WERROR=0 DO_CTFMERGE=0 KERNEL_CONFIGS=RELEASE
            SRCROOT=${CMAKE_CURRENT_SOURCE_DIR} OBJROOT=<BINARY_DIR>
            DSTROOT=${CMAKE_CURRENT_BINARY_DIR}/xnu_install
    INSTALL_COMMAND true # installed during build
    UPDATE_COMMAND "" # prevent constant rebuilds
    USES_TERMINAL_CONFIGURE TRUE
    USES_TERMINAL_BUILD TRUE
)
add_dependencies(xnu xnu_headers libfirehose_kernel)

install(DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/xnu_install/System/Library DESTINATION System COMPONENT BaseSystem)
install(DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/xnu_install/usr/include DESTINATION usr COMPONENT DeveloperTools)
install(DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/xnu_install/usr/libexec DESTINATION usr COMPONENT DeveloperTools)
install(DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/xnu_install/usr/share/man DESTINATION usr/share COMPONENT DeveloperTools)