CMAKE_MINIMUM_REQUIRED( VERSION 3.0.0 FATAL_ERROR )
MESSAGE(STATUS "Building SUPRA")
PROJECT(SUPRA)

MESSAGE( STATUS "CMAKE_GENERATOR: ${CMAKE_GENERATOR}")
MESSAGE( STATUS "CMAKE_CL_64: ${CMAKE_CL_64}")

SET(CMAKE_CXX_STANDARD 11)
SET(CMAKE_CXX_STANDARD_REQUIRED ON)
SET(CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

FIND_PACKAGE( OpenMP REQUIRED)
if(OPENMP_FOUND)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${OpenMP_EXE_LINKER_FLAGS}")
endif()
IF(WIN32)
    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /MP")
ENDIF(WIN32)

# options for modules to include
OPTION(SUPRA_PROFILING             "Add profiling message to logfile"  OFF)
OPTION(SUPRA_DOWNLOAD_SAMPLEDATA   "Whether sample data should be downloaded" ON)
OPTION(SUPRA_CUDA                  "Use cuda in SUPRA"                 ON)
OPTION(SUPRA_BUILD_DOC             "Build doxygen documentation"       OFF)
OPTION(SUPRA_INTERFACE_GRAPHIC     "Build Graphic Interface (QT)"      ON)
OPTION(SUPRA_INTERFACE_GRAPHIC_CMD "Activate Console Output for GUI"   ON)
OPTION(SUPRA_INTERFACE_COMMANDLINE "Build Commandline Interface"       ON)
OPTION(SUPRA_INTERFACE_EXECUTOR    "Build Supra Executor"              ON)
OPTION(SUPRA_INTERFACE_ROS         "Build ROS Interface"               OFF)
OPTION(SUPRA_BEAMFORMER            "Build Software beamformer"         ON)
OPTION(SUPRA_DEVICE_ULTRASOUND_SIM "Build Ultrasound Test"             ON)
OPTION(SUPRA_DEVICE_TRACKING_SIM   "Build Tracker Test"                ON)
OPTION(SUPRA_DEVICE_TRACKING_IGTL  "Build OpenIGTLink Tracking Input"  ON)
OPTION(SUPRA_DEVICE_ULTRASONIX     "Build Ultrasonix Ultrasound"       OFF)
OPTION(SUPRA_DEVICE_IGTL_OUTPUT    "Build OpenIGTLink Output"          ON)
OPTION(SUPRA_DEVICE_CEPHASONICS    "Build Cephasonics Ultrasound"      OFF)
OPTION(SUPRA_DEVICE_ITK_FILE_OUT   "Build ITK file output Interface"   ON)
OPTION(SUPRA_DEVICE_ROS_IMAGE_OUT  "Build ROS image output Interface"  OFF)
SET( SUPRA_DEVICE_ULTRASONIX_V     "5.7"  CACHE STRING "Ultrasonix Version (5.7|6.07)")
IF(WIN32)
	# Ros tracking output is by default activated on windows as it does not add dependencies
	OPTION(SUPRA_DEVICE_TRACKING_ROS   "Build ROS Tracking Input"          ON)
ELSE()
	OPTION(SUPRA_DEVICE_TRACKING_ROS   "Build ROS Tracking Input"          OFF)
ENDIF()

IF(NOT SUPRA_CUDA)
	MESSAGE(WARNING "CUDA has been disabled. This will exclude a number of nodes from the build.")
	IF(SUPRA_BEAMFORMER)
		MESSAGE(WARNING "The software beamformer requires CUDA, I disabled it.")
		SET(SUPRA_BEAMFORMER OFF CACHE BOOL "Build Software beamformer" FORCE)
	ENDIF()
	IF(SUPRA_DEVICE_CEPHASONICS)
		MESSAGE(WARNING "Without CUDA, only the built-in Cephasonics pipeline can be used, but not channel capture")
	ENDIF()
ENDIF()

# external folder
SET( SUPRA_EXTERNALS "${CMAKE_CURRENT_SOURCE_DIR}/external" CACHE PATH "Path to external libs e.g. ultrasonix.")
SET( SUPRA_EXTERNALS_TBB      ""  CACHE PATH "Path to the TBB base dir")
SET( SUPRA_EXTERNALS_TBB_LIBS ""  CACHE PATH "Path to the TBB lib dir")

	
ADD_SUBDIRECTORY(src/SupraLib)
IF(SUPRA_INTERFACE_COMMANDLINE)
ADD_SUBDIRECTORY(src/CommandlineInterface)
ENDIF(SUPRA_INTERFACE_COMMANDLINE)
IF(SUPRA_INTERFACE_EXECUTOR)
ADD_SUBDIRECTORY(src/SupraExecutor)
ENDIF(SUPRA_INTERFACE_EXECUTOR)
IF(SUPRA_INTERFACE_GRAPHIC)
ADD_SUBDIRECTORY(src/GraphicInterface)
ENDIF(SUPRA_INTERFACE_GRAPHIC)
IF(SUPRA_INTERFACE_ROS)
ADD_SUBDIRECTORY(src/RosInterface)
ENDIF(SUPRA_INTERFACE_ROS)
ADD_SUBDIRECTORY(doc)

IF(SUPRA_DOWNLOAD_SAMPLEDATA)
	if(NOT EXISTS ${PROJECT_BINARY_DIR}/data/mockData_linearProbe.zip)
		message(STATUS "Downloading sample data. This may take a while.")
		file(DOWNLOAD
			"http://campar.in.tum.de/files/goeblr/mockData_linearProbe.zip"
			"${PROJECT_BINARY_DIR}/data/mockData_linearProbe.zip"
			SHOW_PROGRESS
		)
	endif()
	
	if(NOT EXISTS ${PROJECT_BINARY_DIR}/data/mockData_extracted)
		message(STATUS "Extracting sample data")
		execute_process(COMMAND cmake -E tar xf mockData_linearProbe.zip 
			WORKING_DIRECTORY "${PROJECT_BINARY_DIR}/data/" )
		execute_process(COMMAND cmake -E touch mockData_extracted
			WORKING_DIRECTORY "${PROJECT_BINARY_DIR}/data/" )
	ENDIF()
	file(COPY config/configDemo.xml DESTINATION "${PROJECT_BINARY_DIR}/data/")
ENDIF(SUPRA_DOWNLOAD_SAMPLEDATA)
