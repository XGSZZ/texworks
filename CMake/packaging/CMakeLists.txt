# Packaging Tasks
# ===============

# Create a CMake script containing packaging tasks specific to Apple. All
# variables of the form `@VARIABLE@` will be replaced, while variables of the
# form `${VARIABLE}` will be left untouched.
IF ( APPLE )
  CONFIGURE_FILE(
    ${PROJECT_SOURCE_DIR}/CMake/packaging/mac/MacPackagingTasks.cmake.in
    ${PROJECT_BINARY_DIR}/CPackPackagingTasks.cmake
    @ONLY
  )
ELSE ()
  # Create a dummy script for other platforms.
  FILE(WRITE ${PROJECT_BINARY_DIR}/CPackPackagingTasks.cmake "")
ENDIF ()

# Execute the packaging tasks. `INSTALL(SCRIPT ...)` is used so that these tasks
# executing during `make install` rather than `make` or `cmake`.
INSTALL(SCRIPT ${PROJECT_BINARY_DIR}/CPackPackagingTasks.cmake)

set(POPPLERDATA_URL https://poppler.freedesktop.org/poppler-data-0.4.10.tar.gz)
set(POPPLERDATA_SHA256 6e2fcef66ec8c44625f94292ccf8af9f1d918b410d5aa69c274ce67387967b30)

if (WIN32)
	include(ExternalProject)
	ExternalProject_Add(poppler-data
		URL ${POPPLERDATA_URL}
		URL_HASH SHA256=${POPPLERDATA_SHA256}
		CMAKE_CACHE_ARGS -DCMAKE_INSTALL_PREFIX:STRING=${CMAKE_BINARY_DIR}
	)
	file(COPY "${CMAKE_SOURCE_DIR}/win32/fonts" DESTINATION "${CMAKE_BINARY_DIR}/share")

	install(DIRECTORY ${CMAKE_BINARY_DIR}/share
		DESTINATION ${CMAKE_INSTALL_PREFIX}
		PATTERN pkgconfig EXCLUDE)
elseif (APPLE)
	include(ExternalProject)
	ExternalProject_Add(poppler-data
		URL ${POPPLERDATA_URL}
		URL_HASH SHA256=${POPPLERDATA_SHA256}
		CMAKE_CACHE_ARGS -DCMAKE_INSTALL_PREFIX:STRING=${CMAKE_BINARY_DIR}/${PROJECT_NAME}.app/Contents/
	)
	file(REMOVE_RECURSE ${CMAKE_BINARY_DIR}/${PROJECT_NAME}.app/Contents/share/pkgconfig)
endif ()
