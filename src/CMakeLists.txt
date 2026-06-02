# Create the main library
add_library(CoinUtils)

# Add the sources
target_sources(CoinUtils
  PRIVATE
    CoinAdjacencyVector.cpp
    CoinAlloc.cpp
    CoinBoundPropagation.cpp
    CoinBronKerbosch.cpp
    CoinBuild.cpp
    CoinCliqueExtender.cpp
    CoinCliqueList.cpp
    CoinCliqueSet.cpp
    CoinConflictGraph.cpp
    CoinCutPool.cpp
    CoinDenseFactorization.cpp
    CoinDenseVector.cpp
    CoinDynamicConflictGraph.cpp
    CoinError.cpp
    CoinFactorization1.cpp
    CoinFactorization2.cpp
    CoinFactorization3.cpp
    CoinFactorization4.cpp
    CoinFileIO.cpp
    CoinFinite.cpp
    CoinIndexedVector.cpp
    CoinKnapsackRow.cpp
    CoinLpIO.cpp
    CoinMessage.cpp
    CoinMessageHandler.cpp
    CoinModel.cpp
    CoinModelUseful.cpp
    CoinModelUseful2.cpp
    CoinMpsIO.cpp
    CoinNodeHeap.cpp
    CoinOddWheelSeparator.cpp
    CoinOslFactorization.cpp
    CoinOslFactorization2.cpp
    CoinOslFactorization3.cpp
    CoinPackedMatrix.cpp
    CoinPackedVector.cpp
    CoinPackedVectorBase.cpp
    CoinParam.cpp
    CoinParamUtils.cpp
    CoinPostsolveMatrix.cpp
    CoinPrePostsolveMatrix.cpp
    CoinPresolveDoubleton.cpp
    CoinPresolveDual.cpp
    CoinPresolveDupcol.cpp
    CoinPresolveEmpty.cpp
    CoinPresolveFixed.cpp
    CoinPresolveForcing.cpp
    CoinPresolveHelperFunctions.cpp
    CoinPresolveImpliedFree.cpp
    CoinPresolveIsolated.cpp
    CoinPresolveMatrix.cpp
    CoinPresolveMonitor.cpp
    CoinPresolvePsdebug.cpp
    CoinPresolveSingleton.cpp
    CoinPresolveSubst.cpp
    CoinPresolveTighten.cpp
    CoinPresolveTripleton.cpp
    CoinPresolveUseless.cpp
    CoinPresolveZeros.cpp
    CoinRational.cpp
    CoinSearchTree.cpp
    CoinShallowPackedVector.cpp
    CoinShortestPath.cpp
    CoinSimpFactorization.cpp
    CoinSnapshot.cpp
    CoinStaticConflictGraph.cpp
    CoinStructuredModel.cpp
    CoinTable.cpp
    CoinTerm.cpp
    CoinWarmStartBasis.cpp
    CoinWarmStartDual.cpp
    CoinWarmStartPrimalDual.cpp
    CoinWarmStartVector.cpp
  PRIVATE
    FILE_SET
      private_headers_src
    TYPE
      HEADERS
    BASE_DIRS
      ${CMAKE_CURRENT_SOURCE_DIR}
    FILES
      CoinOslC.h
      CoinRowType.hpp
      CoinUtilsConfig.h
      config_coinutils_default.h
      config_default.h
      configall_system.h
      configall_system_msc.h
  PRIVATE
    FILE_SET
      private_headers_generated
    TYPE
      HEADERS
    BASE_DIRS
      ${PROJECT_BINARY_DIR}
    FILES
      ${PROJECT_BINARY_DIR}/config.h
      ${PROJECT_BINARY_DIR}/config_coinutils.h
  PUBLIC
    FILE_SET
      public_headers_src
    TYPE
      HEADERS
    BASE_DIRS
      ${CMAKE_CURRENT_SOURCE_DIR}
    FILES
      CoinAlloc.hpp
      CoinBuild.hpp
      CoinDenseVector.hpp
      CoinDistance.hpp
      CoinError.hpp
      CoinFactorization.hpp
      CoinDenseFactorization.hpp
      CoinFileIO.hpp
      CoinFinite.hpp
      CoinFloatEqual.hpp
      CoinHelperFunctions.hpp
      CoinIndexedVector.hpp
      CoinLpIO.hpp
      CoinMessage.hpp
      CoinMessageHandler.hpp
      CoinModel.hpp
      CoinStructuredModel.hpp
      CoinModelUseful.hpp
      CoinMpsIO.hpp
      CoinPackedMatrix.hpp
      CoinPackedVector.hpp
      CoinPackedVectorBase.hpp
      CoinParam.hpp
      CoinPragma.hpp
      CoinPresolveDoubleton.hpp
      CoinPresolveDual.hpp
      CoinPresolveDupcol.hpp
      CoinPresolveEmpty.hpp
      CoinPresolveFixed.hpp
      CoinPresolveForcing.hpp
      CoinPresolveImpliedFree.hpp
      CoinPresolveIsolated.hpp
      CoinPresolveMatrix.hpp
      CoinPresolveMonitor.hpp
      CoinPresolvePsdebug.hpp
      CoinPresolveSingleton.hpp
      CoinPresolveSubst.hpp
      CoinPresolveTighten.hpp
      CoinPresolveTripleton.hpp
      CoinPresolveUseless.hpp
      CoinPresolveZeros.hpp
      CoinRational.hpp
      CoinSearchTree.hpp
      CoinShallowPackedVector.hpp
      CoinSignal.hpp
      CoinSmartPtr.hpp
      CoinSnapshot.hpp
      CoinSort.hpp
      CoinTime.hpp
      CoinTypes.h
      CoinUtility.hpp
      CoinWarmStart.hpp
      CoinWarmStartBasis.hpp
      CoinWarmStartVector.hpp
      CoinWarmStartDual.hpp
      CoinWarmStartPrimalDual.hpp
      CoinAdjacencyVector.hpp
      CoinBronKerbosch.hpp
      CoinCliqueExtender.hpp
      CoinCliqueList.hpp
      CoinCliqueSet.hpp
      CoinConflictGraph.hpp
      CoinCutPool.hpp
      CoinDynamicConflictGraph.hpp
      CoinKnapsackRow.hpp
      CoinBoundPropagation.hpp
      CoinTerm.hpp
      CoinNodeHeap.hpp
      CoinOddWheelSeparator.hpp
      CoinShortestPath.hpp
      CoinStaticConflictGraph.hpp
      CoinColumnType.hpp
      CoinTable.hpp
      CoinOslFactorization.hpp
      CoinSimpFactorization.hpp
  PUBLIC
    FILE_SET
      public_headers_generated
    TYPE
      HEADERS
    BASE_DIRS
      ${PROJECT_BINARY_DIR}/install
    FILES
      ${PROJECT_BINARY_DIR}/install/CoinUtilsConfig.h
)

# We generated a custom config, so we need to privately define HAVE_CONFIG_H
target_compile_definitions(CoinUtils PRIVATE HAVE_CONFIG_H)

# We are building the library, define COINUTILSLIB_BUILD
target_compile_definitions(CoinUtils PRIVATE COINUTILSLIB_BUILD)

# If we are building shared libraries and are on MSVC we define DLL_EXPORT
if(MSVC AND BUILD_SHARED_LIBS)
  target_compile_definitions(CoinUtils PRIVATE DLL_EXPORT)
endif()

# Link the libraries, if they are available
if(COINUTILS_PTHREADS)
  target_include_directories(CoinUtils PRIVATE ${Threads_INCLUDE_DIRS})
  target_link_libraries(CoinUtils PRIVATE ${Threads_LIBRARIES})
endif()

# Determine the type of linkage to third-party libraries
# If static libraries are built, the dependencies must be linked publicly
if(BUILD_SHARED_LIBS)
  set(DEPENDENCY_LINKAGE_TYPE PRIVATE)
else()
  set(DEPENDENCY_LINKAGE_TYPE PUBLIC)
endif()

if(COINUTILS_HAS_ZLIB)
  if(TARGET ZLIB::ZLIB)
    target_link_libraries(CoinUtils ${DEPENDENCY_LINKAGE_TYPE} ZLIB::ZLIB)
  elseif(TARGET ZLIB::ZLIBSTATIC)
    target_link_libraries(CoinUtils ${DEPENDENCY_LINKAGE_TYPE} ZLIB::ZLIBSTATIC)
  else()
    target_include_directories(CoinUtils ${DEPENDENCY_LINKAGE_TYPE} ${ZLIB_INCLUDE_DIRS})
    target_link_libraries(CoinUtils ${DEPENDENCY_LINKAGE_TYPE} ${ZLIB_LIBRARIES})
  endif()
endif()

if(COINUTILS_HAS_BZLIB)
  if(TARGET BZip2::BZip2)
    target_link_libraries(CoinUtils ${DEPENDENCY_LINKAGE_TYPE} BZip2::BZip2)
  else()
    target_include_directories(CoinUtils ${DEPENDENCY_LINKAGE_TYPE} ${BZip2_INCLUDE_DIRS})
    target_link_libraries(CoinUtils ${DEPENDENCY_LINKAGE_TYPE} ${BZip2_LIBRARIES})
  endif()
endif()

if(COINUTILS_HAS_READLINE)
  target_include_directories(CoinUtils ${DEPENDENCY_LINKAGE_TYPE} ${Readline_INCLUDE_DIR})
  target_link_libraries(CoinUtils ${DEPENDENCY_LINKAGE_TYPE} ${Readline_LIBRARY})
endif()

if(COINUTILS_HAS_LAPACK)
  if(TARGET LAPACK::LAPACK)
    target_link_libraries(CoinUtils ${DEPENDENCY_LINKAGE_TYPE} LAPACK::LAPACK)
  else()
    target_include_directories(CoinUtils ${DEPENDENCY_LINKAGE_TYPE} ${LAPACK_INCLUDE_DIRS})
    target_link_libraries(CoinUtils ${DEPENDENCY_LINKAGE_TYPE} ${LAPACK_LIBRARIES})
  endif()
endif()

if(COINUTILS_HAS_GLPK)
  if(TARGET GLPK::GLPK)
    target_link_libraries(CoinUtils ${DEPENDENCY_LINKAGE_TYPE} GLPK::GLPK)
  else()
    target_include_directories(CoinUtils ${DEPENDENCY_LINKAGE_TYPE} ${GLPK_INCLUDE_DIRS})
    target_link_libraries(CoinUtils ${DEPENDENCY_LINKAGE_TYPE} ${GLPK_LIBRARIES})
  endif()
endif()

# We set VERSION and SOVERSION on the target
set_target_properties(CoinUtils PROPERTIES
  VERSION ${PROJECT_VERSION}
  SOVERSION ${PROJECT_VERSION_MAJOR}
)
