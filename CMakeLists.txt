cmake_minimum_required(VERSION 3.27.6)
project(MoreMathFunctions)

# make cache variables for install destinations
include(GNUInstallDirs)
include(FetchContent)


# specify the C++ standard
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)

set(HFC_BASE_DIR "")
if(HERMETIC_FETCHCONTENT_SOURCE_CACHE_DIR)
  set(HFC_BASE_DIR "${HERMETIC_FETCHCONTENT_SOURCE_CACHE_DIR}")
else()
  set(HFC_BASE_DIR "${CMAKE_CURRENT_LIST_DIR}")
endif()

# Provisioning hfc if not already provisioned by parent project / proxy toolchain
include(HermeticFetchContent OPTIONAL RESULT_VARIABLE hfc_included) 
if(NOT hfc_included)
  set(hfc_REPOSITORY https://github.com/tipi-build/hfc)
  set(hfc_REVISION a56de21daf429cf54d9c087524086277480a4eaf) # release v1.0.4

  FetchContent_Populate(
    hermetic_fetchcontent
    GIT_REPOSITORY ${hfc_REPOSITORY}
    GIT_TAG ${hfc_REVISION}
    SOURCE_DIR "${HFC_BASE_DIR}/thirdparty/cache/hfc-${hfc_REVISION}/src"
    SUBBUILD_DIR "${HFC_BASE_DIR}/thirdparty/cache/hfc-${hfc_REVISION}/subbuild"
    BINARY_DIR "${HFC_BASE_DIR}/thirdparty/cache/hfc-${hfc_REVISION}/bin"
  )

  FetchContent_GetProperties(hermetic_fetchcontent)

  message(STATUS "Hermetic FetchContent ${hfc_REVISION} available at '${hermetic_fetchcontent_SOURCE_DIR}'")
  list(APPEND CMAKE_MODULE_PATH "${hermetic_fetchcontent_SOURCE_DIR}/cmake")
  include(HermeticFetchContent)
endif()

FetchContent_Declare(
  "mathlib"
  GIT_REPOSITORY "https://github.com/tipi-build/unit-test-cmake-template-2libs.git"
  GIT_TAG "20f983e688fd77961445608d44bc90df15f38e76"
)

FetchContent_MakeHermetic(
  "mathlib"
  HERMETIC_BUILD_SYSTEM cmake
)

HermeticFetchContent_MakeAvailableAtBuildTime("mathlib")


add_library(More_MathFunctions STATIC More_MathFunctions.cxx)

target_include_directories(More_MathFunctions
                           PUBLIC
                           "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>"
                           "$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>"
)
target_link_libraries(More_MathFunctions MathFunctions::MathFunctions MathFunctionscbrt::MathFunctionscbrt)


install(TARGETS More_MathFunctions
        EXPORT More_MathFunctionsTargets
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
        INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

install(FILES More_MathFunctions.h DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
install(EXPORT More_MathFunctionsTargets
        FILE More_MathFunctionsTargets.cmake
        NAMESPACE More_MathFunctions::
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/More_MathFunctions
)







##"MathFunctions.h", "MathFunctionscbrt.h"
