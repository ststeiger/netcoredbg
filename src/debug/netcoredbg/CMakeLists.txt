# CoreCLR libraries

set(CORECLR_PRODUCT ${CLR_DIR}/bin/Product/${CORECLR_PLATFORM})

if (NOT DBGSHIM_DIR STREQUAL "")
    add_definitions(-DDBGSHIM_DIR="${DBGSHIM_DIR}")
else()
    set(DBGSHIM_LOCATION ${CLR_BIN_DIR}/${CMAKE_SHARED_LIBRARY_PREFIX}dbgshim${CMAKE_SHARED_LIBRARY_SUFFIX})
endif()

add_library(corguids STATIC IMPORTED)
set_target_properties(corguids PROPERTIES IMPORTED_LOCATION ${CLR_BIN_DIR}/lib/${CMAKE_STATIC_LIBRARY_PREFIX}corguids${CMAKE_STATIC_LIBRARY_SUFFIX})

if (NOT WIN32)
    include_directories(${CLR_DIR}/src/pal/inc)
    include_directories(${CLR_DIR}/src/pal/inc/rt)
    include_directories(${CLR_DIR}/src/pal/prebuilt/inc)
endif()
include_directories(${CLR_DIR}/src/inc)
include_directories(${CLR_DIR}/src/debug/inc)
include_directories(${CLR_DIR}/src/debug/shim)
include_directories(${CLR_DIR}/src/dlls/dbgshim)
include_directories(${CLR_DIR}/src/coreclr/hosts/inc)
include_directories(${CLR_DIR}/src/ToolBox/SOS/Strike)

# NetCoreDBG

set(netcoredbg_SRC
    main.cpp
    manageddebugger.cpp
    symbolreader.cpp
    platform.cpp
    breakpoints.cpp
    modules.cpp
    variables.cpp
    typeprinter.cpp
    valuewalk.cpp
    valueprint.cpp
    miprotocol.cpp
    vscodeprotocol.cpp
    frames.cpp
    jmc.cpp
    cputil.cpp
    expr.cpp
    valuewrite.cpp
    logger.cpp)

set(CMAKE_INCLUDE_CURRENT_DIR ON)

if (NOT WIN32)
    add_definitions(-DPAL_STDCPP_COMPAT)
endif()

add_executable(netcoredbg ${netcoredbg_SRC})

if (WIN32)
    target_link_libraries(netcoredbg corguids wsock32 ws2_32)
else()
    target_link_libraries(netcoredbg corguids dl pthread)
endif()

if (CLR_CMAKE_TARGET_TIZEN_LINUX)
    add_definitions(-DDEBUGGER_FOR_TIZEN)
    target_link_libraries(netcoredbg dlog)
endif (CLR_CMAKE_TARGET_TIZEN_LINUX)

install(TARGETS netcoredbg DESTINATION ${CMAKE_INSTALL_PREFIX})
if (DBGSHIM_LOCATION)
    install(FILES ${DBGSHIM_LOCATION} DESTINATION ${CMAKE_INSTALL_PREFIX})
endif()

# SymbolReader

if (BUILD_MANAGED)
    set(SYMBOLREADER_PROJECT ${CMAKE_CURRENT_SOURCE_DIR}/SymbolReader.csproj)
    set(SYMBOLREADER_DLL_NAME SymbolReader.dll)
    set(DOTNET_BUILD_RESULT ${CMAKE_CURRENT_BINARY_DIR}/${SYMBOLREADER_DLL_NAME})

    find_program(DOTNETCLI dotnet PATHS ENV PATH "${CLR_DIR}/Tools/dotnetcli")

    add_custom_command(OUTPUT ${DOTNET_BUILD_RESULT}
      COMMAND ${DOTNETCLI} publish ${SYMBOLREADER_PROJECT} -o ${CMAKE_CURRENT_BINARY_DIR} /p:BaseIntermediateOutputPath=${CMAKE_CURRENT_BINARY_DIR}/obj/
      WORKING_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}"
      DEPENDS "${CMAKE_CURRENT_SOURCE_DIR}/SymbolReader.cs" "${SYMBOLREADER_PROJECT}"
      COMMENT "Compiling ${SYMBOLREADER_DLL_NAME}"
      VERBATIM
    )

    add_custom_target(symbolreader_dll ALL DEPENDS ${DOTNET_BUILD_RESULT})

    # Copy dlls
    set(ROSLYN_DLLS
        Microsoft.CodeAnalysis.dll
        Microsoft.CodeAnalysis.CSharp.dll
        Microsoft.CodeAnalysis.Scripting.dll
        Microsoft.CodeAnalysis.CSharp.Scripting.dll)

    set(DLLS_TO_DEPLOY ${DOTNET_BUILD_RESULT})
    foreach(ITEM ${ROSLYN_DLLS})
        list(APPEND DLLS_TO_DEPLOY "${CMAKE_CURRENT_BINARY_DIR}/${ITEM}")
    endforeach()

    install(FILES ${DLLS_TO_DEPLOY} DESTINATION ${CMAKE_INSTALL_PREFIX})
endif()
