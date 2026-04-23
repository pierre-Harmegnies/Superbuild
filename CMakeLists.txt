cmake_minimum_required(VERSION 3.10)
project(Superbuild_Demo NONE) # NONE car le superbuild lui-même n'a pas de code C++

include(ExternalProject)

# Chemin où tout sera installé (libs, includes, bins)
set(SB_INSTALL_DIR ${CMAKE_CURRENT_BINARY_DIR}/install)

# Macro pour automatiser l'ajout d'une bibliothèque
macro(bundle_lib NOM GIT_URL)
    # Chemin potentiel pour une source locale (dossier d'à côté)
    set(LOCAL_SRC "${CMAKE_CURRENT_SOURCE_DIR}/../${NOM}")
    
    # Si MaLib/CMakeLists.txt existe à côté de Superbuild, on utilise les sources locales
    if(EXISTS "${LOCAL_SRC}/CMakeLists.txt")
        message(STATUS "Mode Local : Utilisation de ${LOCAL_SRC}")
        set(FETCH_ARGS SOURCE_DIR "${LOCAL_SRC}")
    else()
        # Sinon, on configure le téléchargement depuis GitHub dans build/src/
        message(STATUS "Mode Fetch : Telechargement de ${NOM} depuis ${GIT_URL}")
        set(FETCH_ARGS 
            GIT_REPOSITORY "${GIT_URL}"
            GIT_TAG main
            SOURCE_DIR "${CMAKE_BINARY_DIR}/src/${NOM}"
        )
    endif()

    # Définition du projet externe
    ExternalProject_Add(${NOM}
        ${FETCH_ARGS}
        INSTALL_DIR ${SB_INSTALL_DIR}
        CMAKE_ARGS 
            -DCMAKE_INSTALL_PREFIX=<INSTALL_DIR> # Où installer le projet une fois compilé
            -DCMAKE_PREFIX_PATH=${SB_INSTALL_DIR} # Où chercher les dépendances (ex: MaLib1)
            -DCMAKE_INSTALL_RPATH=${SB_INSTALL_DIR}/lib # Chemin de recherche des .so à l'exécution
            -DCMAKE_BUILD_WITH_INSTALL_RPATH=ON # Appliquer le RPATH dès le build
    )
endmacro()

# 1. Définition des bibliothèques de base
bundle_lib(MaLib1 https://github.com/pierre-Harmegnies/MaLib1.git)
bundle_lib(MaLib2 https://github.com/pierre-Harmegnies/MaLib2.git)

# 2. Lib 3 (dépend de MaLib1)
bundle_lib(MaLib3_Python https://github.com/pierre-Harmegnies/MaLib3_Python.git)
# Indique à CMake que MaLib3 doit attendre que MaLib1 soit configurée/installée
ExternalProject_Add_StepDependencies(MaLib3_Python configure MaLib1)

# 3. Application Finale (dépend de tout le monde)
bundle_lib(MonAppFinale https://github.com/pierre-Harmegnies/MonAppFinale.git)
ExternalProject_Add_StepDependencies(MonAppFinale configure MaLib1 MaLib2 MaLib3_Python)

# 4. Cible personnalisée pour tout nettoyer (Deep Clean)
add_custom_target(clean_all
    COMMAND ${CMAKE_COMMAND} -E remove_directory ${SB_INSTALL_DIR} # Supprime l'install
    COMMAND ${CMAKE_COMMAND} -E remove_directory ${CMAKE_BINARY_DIR}/src # Supprime les sources fetchées
    # Supprime les dossiers de stamps (mémoire du build) pour permettre un restart propre
    COMMAND rm -rf ${CMAKE_BINARY_DIR}/MaLib1-prefix
    COMMAND rm -rf ${CMAKE_BINARY_DIR}/MaLib2-prefix
    COMMAND rm -rf ${CMAKE_BINARY_DIR}/MaLib3_Python-prefix
    COMMAND rm -rf ${CMAKE_BINARY_DIR}/MonAppFinale-prefix
    COMMAND ${CMAKE_COMMAND} -E echo "Nettoyage complet effectue - install, src et stamps supprimes."
    COMMENT "Nettoyage profond pour permettre un redemarrage sans re-configure"
)
