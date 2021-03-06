set (TCTFB_SRCS video.cpp)
set (TCTFB_CUDA_SRCS ${TCTFB_SRCS} ${CMAKE_CURRENT_SOURCE_DIR}/../cuda/cuda_kernel.cu ${CMAKE_CURRENT_SOURCE_DIR}/../cuda/time_interval.cu)

include_directories (
  ${PROJECT_SOURCE_DIR}
  ${PROJECT_SOURCE_DIR}/atomic
  ${CMAKE_CURRENT_SOURCE_DIR}
  ${CMAKE_CURRENT_SOURCE_DIR}/..
  ${CMAKE_CURRENT_SOURCE_DIR}/../cuda
  ${CMAKE_CURRENT_SOURCE_DIR}/../cuda/cuda_utils
  )

#set( OpenCV_DIR /opt/local/lib/cmake  )
#set( OpenCV_DIR C:/opencv/build )

FIND_PACKAGE( OpenCV )

FIND_FILE (HIGHGUI "opencv/highgui.h" PATHS "${OpenCV_CONFIG_PATH}/include" "/usr/include"
"/opt/local/include" "$ENV{INCLUDE}" DOC "-- Checking for highgui.h" )

if (${HIGHGUI} MATCHES "HIGHGUI-NOTFOUND")
   message (WARNING "OpenCV highgui.h not found")
endif (${HIGHGUI} MATCHES "HIGHGUI-NOTFOUND")

if ((OpenCV_FOUND) AND NOT (${HIGHGUI} MATCHES "HIGHGUI-NOTFOUND"))
  #message(STATUS "Using ${OpenCV_LIBS}")
  add_executable (tctfb_ff ${TCTFB_SRCS})
  set_target_properties (tctfb_ff PROPERTIES COMPILE_DEFINITIONS "TIME;SEQ_DETECTION")
  target_link_libraries (tctfb_ff ${CMAKE_THREAD_LIBS_INIT})
  TARGET_LINK_LIBRARIES( tctfb_ff ${OpenCV_LIBS} )



  #CUDA variants
  #allowed flags: FF_WITH_CUDA_PINNED_MEMORY
  FIND_PACKAGE(CUDA)

  if(CUDA_FOUND)
    #link_directories("/usr/local/cuda/lib")
    include(FindCUDA)
    set(CUDA_NVCC_FLAGS "--use_fast_math")
  
    cuda_add_executable ( tctfb_cuda ${TCTFB_CUDA_SRCS} )
    #target_link_libraries ( tctfb_cuda "cudart" )
    target_link_libraries( tctfb_cuda ${OpenCV_LIBS} )
    set_target_properties ( tctfb_cuda PROPERTIES COMPILE_DEFINITIONS "FF_WITH_CUDA;TIME;AVG_TERMINATION" )
    #set_target_properties ( tctfb_cuda PROPERTIES COMPILE_FLAGS "-O2" )

  else(CUDA_FOUND)
    message (WARNING " CUDA not found: can't configure CUDA-enabled tctfb (video denoiser)")
  endif(CUDA_FOUND)
else ((OpenCV_FOUND) AND NOT (${HIGHGUI} MATCHES "HIGHGUI-NOTFOUND"))
  message (WARNING " OpenCV or opencv/highgui.h not found: can't configure tctfb (video denoiser)")
endif  ((OpenCV_FOUND) AND NOT (${HIGHGUI} MATCHES "HIGHGUI-NOTFOUND"))
