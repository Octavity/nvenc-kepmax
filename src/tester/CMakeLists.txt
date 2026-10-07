cmake_minimum_required(VERSION 3.28...3.30)

find_package(FFnvcodec 11 REQUIRED)

add_executable(${CMAKE_PROJECT_NAME}-test)

target_sources(${CMAKE_PROJECT_NAME}-test PRIVATE tester.cpp)
target_link_libraries(${CMAKE_PROJECT_NAME}-test PUBLIC FFnvcodec::FFnvcodec)

# The plugin spawns this helper at load time to probe NVENC capabilities. It is
# located at runtime with obs_module_file(), so it must be installed into the
# plugin's data directory (next to the locale files) on every platform.
if(WIN32)
  install(TARGETS ${CMAKE_PROJECT_NAME}-test RUNTIME DESTINATION "${CMAKE_PROJECT_NAME}/data")
elseif(APPLE)
  install(
    TARGETS ${CMAKE_PROJECT_NAME}-test
    RUNTIME DESTINATION "${CMAKE_PROJECT_NAME}.plugin/Contents/Resources"
  )
else()
  install(
    TARGETS ${CMAKE_PROJECT_NAME}-test
    RUNTIME DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/obs/obs-plugins/${CMAKE_PROJECT_NAME}
  )
endif()
