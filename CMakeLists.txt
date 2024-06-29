cmake_minimum_required(VERSION 3.16)
project(CacaoAudioLoader)

message("========= OGG ========")
add_subdirectory(ogg)

message("======== OPUS ========")
add_subdirectory(opus)

function(so_fullname result basename)
	set(${result} "${CMAKE_SHARED_LIBRARY_PREFIX}${basename}${CMAKE_SHARED_LIBRARY_SUFFIX}" PARENT_SCOPE)
endfunction()
function(lib_fullname result basename)
	set(${result} "${CMAKE_STATIC_LIBRARY_PREFIX}${basename}${CMAKE_STATIC_LIBRARY_SUFFIX}" PARENT_SCOPE)
endfunction()
set(OGG_DIR "${CMAKE_BINARY_DIR}/ogg")
lib_fullname(OGG_LIB "ogg")
set(OGG_LIBRARY "${OGG_DIR}/${OGG_LIB}")
set(OGG_INCLUDE_DIR "${CMAKE_SOURCE_DIR}/ogg/include")

macro(find_package)
	_find_package(${ARGV})
	if("${ARGV0}" STREQUAL "Ogg")
		set(OGG_INCLUDE_DIRS ${OGG_INCLUDE_DIRS} "${CMAKE_SOURCE_DIR}/ogg/include" "${OGG_DIR}/include")
        set(OGG_FOUND TRUE)
	endif()
endmacro()

message("======= VORBIS =======")
add_subdirectory(vorbis)

list(PREPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR}/cmake")

message("====== OPUSFILE ======")
# Capture the original target_compile_options function
function(_original_target_compile_options target)
  _target_compile_options(${ARGV})
endfunction()

# Override target_compile_options with our custom implementation
macro(target_compile_options target vis)
  set(options ${ARGN})

  # Initialize a list to store the modified compile options
  set(new_compile_options)

  # Iterate over the compile options
  foreach(option IN LISTS options)
    # Check if the option contains "-std=c89" as a substring and replace it with "-std=c99"
    string(REPLACE "-std=c89" "-std=c99" modified_option ${option})
    list(APPEND new_compile_options ${modified_option})
  endforeach()

  # Call the original target_compile_options with the modified arguments
  _original_target_compile_options(${target} ${vis} ${new_compile_options})
endmacro()
add_subdirectory(opusfile)
