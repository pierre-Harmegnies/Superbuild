cmake_minimum_required(VERSION 3.10)
project(Superbuild_Demo NONE)

include(ExternalProject)

# Prefix d'installation commun
set(SB_INSTALL_DIR ${CMAKE_CURRENT_BINARY_DIR}/install)

# Macro simplifiée pour la démo locale
macro(bundle_lib NOM GIT_URL)
    set(LOCAL_SRC "${CMAKE_CURRENT_SOURCE_DIR}/../${NOM}")
    
    if(EXISTS "${LOCAL_SRC}/CMakeLists.txt")
        message(STATUS "Found local sources for ${NOM} at ${LOCAL_SRC}")
        set(FETCH_ARGS SOURCE_DIR "${LOCAL_SRC}")
    else()
        message(STATUS "${NOM} local sources not found or incomplete. Fetching from ${GIT_URL}")
        set(FETCH_ARGS 
            GIT_REPOSITORY "${GIT_URL}"
            GIT_TAG main
            SOURCE_DIR "${CMAKE_BINARY_DIR}/src/${NOM}"
        )
    endif()

    ExternalProject_Add(${NOM}
        ${FETCH_ARGS}
        INSTALL_DIR ${SB_INSTALL_DIR}
        CMAKE_ARGS 
            -DCMAKE_INSTALL_PREFIX=<INSTALL_DIR>
            -DCMAKE_PREFIX_PATH=${SB_INSTALL_DIR}
    )
endmacro()

# 1. Compilation des bases
bundle_lib(MaLib1 https://github.com/pierre-Harmegnies/MaLib1.git)
bundle_lib(MaLib2 https://github.com/pierre-Harmegnies/MaLib2.git)

# 2. Lib 3 (Depend de Lib 1)
bundle_lib(MaLib3_Python https://github.com/pierre-Harmegnies/MaLib3_Python.git)
ExternalProject_Add_StepDependencies(MaLib3_Python configure MaLib1)

# 3. Application (Depend de TOUT)
bundle_lib(MonAppFinale https://github.com/pierre-Harmegnies/MonAppFinale.git)
ExternalProject_Add_StepDependencies(MonAppFinale configure MaLib1 MaLib2 MaLib3_Python)
