cmake_minimum_required(VERSION 3.31)
project(C3CMake)

block(
  PROPAGATE
  C3CMake_import_sources
  C3CMake_import_sources_glob
  C3CMake_import_sources_glob_absolute
)
  macro(get_target_source_directory)
    get_target_property(target_source_directory ${target} SOURCE_DIR)
  endmacro()

  function(C3CMake_message message #[[mode]])
    set(prefix "[C3CMake]\t")
    if(NOT ARGN)
      set(mode STATUS)
    elseif(ARGC EQUAL 2)
      set(mode ${ARGV1})
    else()
      message(FATAL_ERROR "unexpected arguments passed to C3CMake_message")
    endif()
    message(${mode} ${prefix} ${message})
  endfunction()

  macro(message_list label list)
    C3CMake_message("${label} :")
    foreach(item ${${list}})
      C3CMake_message("\t- ${item}")
    endforeach()
  endmacro()

  macro(append_to_configure_depends path)
    set_property(DIRECTORY ${target_source_directory} APPEND PROPERTY CMAKE_CONFIGURE_DEPENDS ${path})
    block()
      get_property(configure_depends DIRECTORY ${target_source_directory} PROPERTY CMAKE_CONFIGURE_DEPENDS)
      message_list("updated CMAKE_CONFIGURE_DEPENDS for directory ${target_source_directory}" configure_depends)
    endblock()
  endmacro()

  function(C3CMake_import_sources target sources)
    get_target_source_directory()
    C3CMake_message("target source directory : ${target_source_directory}")
    get_target_property(target_binary_directory ${target} BINARY_DIR)
    C3CMake_message("target binary directory : ${target_binary_directory}")
    set(compilation_output_directory ${target_binary_directory}/c3)
    C3CMake_message("compilation output directory : ${compilation_output_directory}")
    if(EXISTS ${compilation_output_directory})
      if(IS_DIRECTORY ${compilation_output_directory})
        file(REMOVE_RECURSE ${compilation_output_directory})
        C3CMake_message("removed pre-existing compilation output directory")
      else()
        C3CMake_message("compilation output directory path (${compilation_output_directory}) is an existing file" FATAL_ERROR)
      endif()
    endif()
    message_list("c3 sources" sources)
    if(MINGW)
      set(c3c_target_option "--target" "mingw-x64")
    endif()
    set(c3c_options
      -vv --print-output ${c3c_target_option}
      --obj-out ${compilation_output_directory}
      compile-only ${sources}
    )
    block()
      set(options_string "")
      foreach(option ${c3c_options})
        string(APPEND options_string ${option} " ")
      endforeach()
      C3CMake_message("compiling c3 sources with options : ${options_string}")
    endblock()
    execute_process(
      COMMAND c3c ${c3c_options}
      WORKING_DIRECTORY ${target_source_directory}
      COMMAND_ERROR_IS_FATAL ANY
    )
    C3CMake_message("done compiling c3 sources")
    file(GLOB_RECURSE objects ${compilation_output_directory}/*.obj ${compilation_output_directory}/*.o)
    message_list("c3 compilation output" objects)
    target_sources(${target} PRIVATE ${objects})
    if(WIN32)
      target_link_libraries(${target} PRIVATE Dbghelp.lib)
    endif()
  endfunction()

  function(C3CMake_import_sources_glob target sources_path)
    get_target_source_directory()
    C3CMake_import_sources_glob_absolute(${target} ${target_source_directory}/${sources_path})
  endfunction()

  function(C3CMake_import_sources_glob_absolute target sources_path)
    C3CMake_message("globbing sources from path : ${sources_path}")
    file(GLOB_RECURSE sources ${sources_path}/*.c3)
    append_to_configure_depends(${sources_path})
    C3CMake_import_sources(${target} "${sources}")
  endfunction()
endblock()