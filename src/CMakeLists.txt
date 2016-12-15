
set (LIBCMAES_SOURCES
acovarianceupdate.cc
cmaparameters.cc
cmastopcriteria.cc
covarianceupdate.cc
esostrategy.cc
pwq_bound_strategy.cc
vdcmaupdate.cc
bipopcmastrategy.cc
cmasolutions.cc
cmastrategy.cc
errstats.cc
ipopcmastrategy.cc
surrogatestrategy.cc)

set (LIBCMAES_HEADERS
acovarianceupdate.h  cmastopcriteria.h   errstats.h         noboundstrategy.h     surrcmaes.h
bipopcmastrategy.h   cmastrategy.h       esoptimizer.h      opti_err.h            surrogatestrategy.h
candidate.h          contour.h           esostrategy.h      parameters.h          vdcmaupdate.h
cmaes.h              covarianceupdate.h  genopheno.h        pli.h
cmaparameters.h      eigenmvn.h          ipopcmastrategy.h  pwq_bound_strategy.h
cmasolutions.h       eo_matrix.h         llogging.h         scaling.h)


include_directories (${CMAKE_CURRENT_SOURCE_DIR})

add_library (cmaes ${LIBCMAES_SOURCES})

set_target_properties (cmaes PROPERTIES SOVERSION 0)
set_target_properties (cmaes PROPERTIES VERSION 0.0.0)
set_target_properties (cmaes PROPERTIES POSITION_INDEPENDENT_CODE ON)

install ( TARGETS cmaes
          EXPORT  LibcmaesLibraryDepends
          RUNTIME DESTINATION ${RELATIVE_INSTALL_BIN_DIR}
          LIBRARY DESTINATION ${RELATIVE_INSTALL_LIB_DIR}
          ARCHIVE DESTINATION ${RELATIVE_INSTALL_LIB_DIR}
        )

install (FILES ${LIBCMAES_HEADERS} DESTINATION ${RELATIVE_INSTALL_INCLUDE_DIR}/libcmaes)
