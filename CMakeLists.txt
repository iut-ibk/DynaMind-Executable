FILE(GLOB EXE_CPPS ./src/*.cpp)

IF(DYNAMIND_TOOLBOX_BUILD_DIR)
    SET(LIBRARY_OUTPUT_PATH ${DYNAMIND_TOOLBOX_BUILD_DIR})
    SET(EXECUTABLE_OUTPUT_PATH ${DYNAMIND_TOOLBOX_BUILD_DIR})
ELSE()
    SET(LIBRARY_OUTPUT_PATH ${dynamind-gui_BINARY_DIR})
    SET(EXECUTABLE_OUTPUT_PATH ${dynamind-gui_BINARY_DIR})
ENDIF()

FIND_PATH(DYNAMINDDEV DynaMindDevCmake.txt
            PATHS   $ENV{DYNAMIND_DIR}/include
            PATH_SUFFIXES dynamindcore/CMake)


IF(DYNAMIND_INCLUDE_DIR)
        MESSAGE(STATUS "Compiled from toolbox")
        MESSAGE(STATUS "DYNAMIND_INCLUDE_DIR ${DYNAMIND_INCLUDE_DIR}")
        SET (DYNAMIND_CORE_LIBRARY dynamindcore)
        SET (DYNAMIND_TOOLBOX_LIBRARY dynamindtoolbox)
        SET (DYNAMIND_EXTENSIONS_LIBRARY dynamind-extensions)
        MESSAGE(STATUS "${DYNAMIND_CORE_LIBRARY}")
ELSE()
        MESSAGE(STATUS "Compiled from stand alone")
        FIND_PATH(DYNAMINDDEV DynaMindDevCmake.txt
        PATHS $ENV{DYNAMIND_DIR}/include
        PATH_SUFFIXES dynamindcore/CMake)

	IF(DYNAMINDDEV)
		INCLUDE(${DYNAMINDDEV}/DynaMindDevCmake.txt)
	ELSE()
		MESSAGE(FATAL_ERROR "Could not find dynamind. Maybe not installed ?")
	ENDIF()
	ELSE()
		MESSAGE(STATUS "Compiled from toolbox")
		MESSAGE(STATUS "DYNAMIND_INCLUDE_DIR ${DYNAMIND_INCLUDE_DIR}")
		SET (DYNAMIND_CORE_LIBRARY dynamindcore)
		MESSAGE(STATUS "${DYNAMIND_CORE_LIBRARY}")
ENDIF()

IF(USEQT5)
    MESSAGE(STATUS "Using Qt5 for executable")
    FIND_PACKAGE(Qt5Core)
ELSE()
    MESSAGE(STATUS "Using Qt4 for executable")
    FIND_PACKAGE(Qt4 COMPONENTS QtCore QtXml REQUIRED)
ENDIF()

SET(Boost_USE_STATIC_LIBS        ON)
SET(Boost_USE_MULTITHREADED      ON)
FIND_PACKAGE(Boost COMPONENTS system program_options REQUIRED)
#ADD_DEFINITIONS(-DQT_NO_KEYWORDS)

FIND_PACKAGE(OpenMP)

IF(OPENMP_FOUND)
    SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
    SET(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${OpenMP_EXE_LINKER_FLAGS}")
ENDIF()

IF(Boost_FOUND)
    INCLUDE_DIRECTORIES(${Boost_INCLUDE_DIRS})
ENDIF()

IF(NOT USEQT5)
    INCLUDE(${QT_USE_FILE})
    INCLUDE_DIRECTORIES(${QT_QTCORE_INCLUDE_DIR})
ENDIF()

INCLUDE_DIRECTORIES(${DYNAMIND_INCLUDE_DIR})

# Building Executable
ADD_EXECUTABLE(dynamind ${EXE_CPPS})
IF(USEQT5)
    TARGET_LINK_LIBRARIES(dynamind ${DYNAMIND_CORE_LIBRARY} ${Boost_PROGRAM_OPTIONS_LIBRARY} ${PYTHON_LIBRARIES})
ELSE()
    TARGET_LINK_LIBRARIES(dynamind ${DYNAMIND_CORE_LIBRARY} ${QT_QTMAIN_LIBRARY} ${Boost_PROGRAM_OPTIONS_LIBRARY} ${QT_LIBRARIES} ${PYTHON_LIBRARIES})
ENDIF()

INSTALL(TARGETS dynamind
		DESTINATION "bin"
		PERMISSIONS OWNER_WRITE OWNER_READ OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE)
