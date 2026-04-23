cmake_minimum_required(VERSION 3.10)
project(Superbuild_Demo NONE)

include(ExternalProject)

# Prefix d'installation commun
set(SB_INSTALL_DIR ${CMAKE_CURRENT_BINARY_DIR}/install)

# Macro simplifiée pour la démo locale
macro(bundle_lib NOM REP_SOURCE)
    ExternalProject_Add(${NOM}
        SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/../${REP_SOURCE}
        INSTALL_DIR ${SB_INSTALL_DIR}
        CMAKE_ARGS 
            -DCMAKE_INSTALL_PREFIX=<INSTALL_DIR>
            -DCMAKE_PREFIX_PATH=<INSTALL_DIR>
    )
endmacro()

# 1. Compilation des bases
bundle_lib(MaLib1 MaLib1)
bundle_lib(MaLib2 MaLib2)

# 2. Lib 3 (Depend de Lib 1)
bundle_lib(MaLib3_Python MaLib3_Python)
ExternalProject_Add_StepDependencies(MaLib3_Python configure MaLib1)

# 3. Application (Depend de TOUT)
bundle_lib(MonAppFinale MonAppFinale)
ExternalProject_Add_StepDependencies(MonAppFinale configure MaLib1 MaLib2 MaLib3_Python)
