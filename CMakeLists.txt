cmake_minimum_required(VERSION 2.8.12)

set(POD_NAME husky)
include(cmake/pods.cmake)
include(cmake/lcmtypes.cmake)
lcmtypes_build()
