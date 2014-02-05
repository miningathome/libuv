cmake_minimum_required(VERSION 2.8)

project(libuv C)

option(BUILD_SHARED_LIBS "Build shared libraries" ON)

# make visual studio and xcode output libs and binaries without the configuration name (Release/Debug)
include (${CMAKE_CURRENT_SOURCE_DIR}/cmake/clean_slate.cmake)

# Common for all platforms
set(sources 
	src/fs-poll.c
	src/inet.c
	src/uv-common.c
	src/version.c)

# i'm lazy.. copy all
set(public_headers
	# include/uv-bsd.h
	# include/uv-darwin.h
	# include/uv-linux.h
	# include/uv-sunos.h
	# include/uv-unix.h
	include/uv-errno.h
	include/uv.h)

set(headers
	${public_headers})

if (WIN32)
	if (BUILD_SHARED_LIBS)
		add_definitions(-DDLL_EXPORT)
	endif()

	add_definitions(-D_GNU_SOURCE)
	add_definitions(-D_WIN32_WINNT=0x0600)
	add_definitions(-D_CRT_SECURE_NO_WARNINGS)
	add_definitions(-D_CRT_NONSTDC_NO_DEPRECATE)

	# Disable warning C4133
	add_definitions("/wd4133")

	set(sources
		${sources}
		src/win/async.c
		src/win/core.c
		src/win/dl.c
		src/win/error.c
		src/win/fs-event.c
		src/win/fs.c
		src/win/getaddrinfo.c
		src/win/handle.c
		src/win/loop-watcher.c
		src/win/pipe.c
		src/win/poll.c
		src/win/process-stdio.c
		src/win/process.c
		src/win/req.c
		src/win/signal.c
		src/win/stream.c
		src/win/tcp.c
		src/win/thread.c
		src/win/threadpool.c
		src/win/timer.c
		src/win/tty.c
		src/win/udp.c
		src/win/util.c
		src/win/winapi.c
		src/win/winsock.c)

	set(public_headers
		${public_headers}
		include/uv-win.h)

	set(platformlibs
		advapi32 iphlpapi psapi shell32 ws2_32)
endif()

include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/src)

if (BUILD_SHARED_LIBS)
	add_definitions(-DBUILDING_UV_SHARED)
	set(sharedlib SHARED)
endif()

add_library(libuv ${sharedlib} ${sources} ${headers})
target_link_libraries(libuv ${platformlibs})

set_target_properties(libuv PROPERTIES
	PUBLIC_HEADER "${public_headers}")

install(TARGETS libuv
	RUNTIME DESTINATION bin
	ARCHIVE DESTINATION lib
	LIBRARY DESTINATION lib
	FRAMEWORK DESTINATION "Library/Frameworks"
	PUBLIC_HEADER DESTINATION include)
