cmake_minimum_required(VERSION 2.8.3)
project(opencv_colcon)

include(ExternalProject)

SET(CONTRIB_NAME "opencv_contrib")
ExternalProject_Add(${CONTRIB_NAME}
  URL https://github.com/opencv/opencv_contrib/archive/4.5.1.zip
  URL_MD5 ddb4f64d6cf31d589a8104655d39c99b
  SOURCE_DIR ${CONTRIB_NAME}_src
  UPDATE_COMMAND ""
  CONFIGURE_COMMAND ""
  BUILD_COMMAND ""
  INSTALL_COMMAND ""
)

SET(OPENCV_SRC_PATH "opencv_src")
SET(OPENCV_BUILD_PATH "opencv_build")
ExternalProject_Add(opencv_src
  DEPENDS ${CONTRIB_NAME}
  URL https://github.com/opencv/opencv/archive/4.5.1.zip
  URL_MD5 cc13d83c3bf989b0487bb3798375ee08
  UPDATE_COMMAND ""
  SOURCE_DIR ${OPENCV_SRC_PATH}
  BINARY_DIR ${OPENCV_BUILD_PATH}
  PATCH_COMMAND patch -p0 < ${CMAKE_SOURCE_DIR}/fix_python_discovery.patch &&
                patch -p1 < ${CMAKE_CURRENT_SOURCE_DIR}/rename_unordered_map_definition.patch
  CONFIGURE_COMMAND cd ../${OPENCV_BUILD_PATH} && cmake
    -DOPENCV_EXTRA_MODULES_PATH=../${CONTRIB_NAME}_src/modules .
    -DWITH_TBB=ON
    -DBUILD_opencv_python3=ON
    -DWITH_PROTOBUF=OFF # Disable custom build of protobuf, since starting with OpenCV 3.4.2 a custom version of
                        # protobuf is always pulled and compiled, even if no enabled modules use it.
    -DWITH_V4L=ON
    -DINSTALL_C_EXAMPLES=OFF
    -DINSTALL_PYTHON_EXAMPLES=OFF
    -DBUILD_EXAMPLES=OFF
    -DWITH_QT=OFF # Needed by clang under Ubuntu 14.04 and GTK_WIDGET(cvGetWindowHandle(...)) with gcc (image_view)
    -DWITH_OPENGL=ON
    -DWITH_VTK=ON
    -DENABLE_PRECOMPILED_HEADERS=OFF
    -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX}
    -DBUILD_SHARED_LIBS=ON
    -DWITH_CUDA=OFF
    -DWITH_OPENCL=OFF
    -DBUILD_TESTS=OFF
    -DBUILD_opencv_ts=OFF
    -DBUILD_TESTS=OFF
    # opencv_contrib packages
    -DBUILD_opencv_dnn=OFF # Pulls in the system protobuf as a dependency!
    -DBUILD_opencv_dnns_easily_fooled=OFF
    -DBUILD_opencv_cnn_3dobj=OFF
    -DBUILD_opencv_aruco=ON
    -DBUILD_opencv_bgsegm=OFF
    -DBUILD_opencv_bioinspired=OFF
    -DBUILD_opencv_ccalib=OFF
    -DBUILD_opencv_contrib_world=OFF
    -DBUILD_opencv_datasets=OFF
    -DBUILD_opencv_dpm=OFF
    -DBUILD_opencv_face=OFF
    -DBUILD_opencv_fuzzy=OFF
    -DBUILD_opencv_freetype=OFF
    -DBUILD_opencv_hdf=OFF
    -DBUILD_opencv_line_descriptor=ON
    -DBUILD_opencv_matlab=OFF
    -DBUILD_opencv_optflow=OFF
    -DBUILD_opencv_plot=OFF
    -DBUILD_opencv_reg=OFF
    -DBUILD_opencv_rgbd=ON
    -DBUILD_opencv_saliency=ON
    -DBUILD_opencv_sfm=OFF
    -DBUILD_opencv_stereo=OFF
    -DBUILD_opencv_structured_light=OFF
    -DBUILD_opencv_surface_matching=OFF
    -DBUILD_opencv_text=OFF
    -DBUILD_opencv_tracking=OFF
    -DBUILD_opencv_xfeatures2d=ON
    -DBUILD_opencv_ximgproc=ON
    -DBUILD_opencv_xobjdetect=OFF
    -DBUILD_opencv_nonfree=OFF
    -DBUILD_opencv_xphoto=OFF ../${OPENCV_SRC_PATH}
  BUILD_COMMAND cd ../${OPENCV_BUILD_PATH} && make -j12
  INSTALL_COMMAND cd ../${OPENCV_BUILD_PATH} && make install -j12
)

include(GNUInstallDirs)

install(DIRECTORY include/opencv4
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
        FILES_MATCHING PATTERN "*.h"
                       PATTERN "*.hpp")
install(DIRECTORY lib/
        DESTINATION ${CMAKE_INSTALL_LIBDIR}
        FILES_MATCHING PATTERN "libopencv*")
# install(DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/${OPENCV_BUILD_PATH}/share/opencv4
#         DESTINATION ${CMAKE_INSTALL_DATADIR})
set(OPENCV_INSTALLED_EXECUTABLES
    opencv_annotation
    opencv_interactive-calibration
    opencv_version
    opencv_visualisation
)

foreach(installed_file ${OPENCV_INSTALLED_EXECUTABLES})
  # Need to rename binaries as they will conflict with the binaries from
  # the ROS OpenCV package otherwise.
  string(REPLACE opencv opencv_colcon new_file_name ${installed_file} )
  install(FILES ${CMAKE_CURRENT_BINARY_DIR}/${OPENCV_BUILD_PATH}/bin/${installed_file}
          RENAME ${new_file_name}
          PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE
          DESTINATION ${CMAKE_INSTALL_BINDIR})
endforeach()
