add_library(amgcl_example INTERFACE)
target_include_directories(amgcl_example INTERFACE
    ${CMAKE_CURRENT_SOURCE_DIR}/../tests
    ${Boost_INCLUDE_DIRS}
    )

target_link_libraries(amgcl_example INTERFACE
    amgcl
    ${Boost_PROGRAM_OPTIONS_LIBRARY}
    )

function(add_amgcl_example EXAMPLE SOURCES)
    add_executable(${EXAMPLE} ${SOURCES})
    target_link_libraries(${EXAMPLE} amgcl_example)
endfunction()

add_amgcl_example(mm2bin mm2bin.cpp)
add_amgcl_example(solver solver.cpp)
add_amgcl_example(solver_complex solver_complex.cpp)
add_amgcl_example(crs_builder crs_builder.cpp)
add_amgcl_example(block_crs block_crs.cpp)
add_amgcl_example(schur_pressure_correction schur_pressure_correction.cpp)
add_amgcl_example(cpr cpr.cpp)
add_amgcl_example(cpr_drs cpr_drs.cpp)
add_amgcl_example(custom_adapter custom_adapter.cpp)

add_amgcl_example(ublas ublas.cpp)
target_link_libraries(ublas ${Boost_SERIALIZATION_LIBRARY})

if (TARGET eigen_target)
    add_amgcl_example(solver_eigen solver.cpp)
    target_link_libraries(solver_eigen eigen_target)
    target_compile_definitions(solver_eigen PRIVATE SOLVER_BACKEND_EIGEN)
endif()

if (TARGET blaze_target)
    add_amgcl_example(solver_blaze solver.cpp)
    target_link_libraries(solver_blaze blaze_target)
    target_compile_definitions(solver_blaze PRIVATE SOLVER_BACKEND_BLAZE)
endif()

add_amgcl_example(call_lib call_lib.cpp)
target_link_libraries(call_lib libamgcl)

#----------------------------------------------------------------------------
# The HPX backend example
#----------------------------------------------------------------------------
find_package(HPX QUIET)
if (HPX_FOUND)
    add_hpx_executable(solver_hpx SOURCES solver_hpx.cpp)
    hpx_setup_target(solver_hpx_exe DEPENDENCIES amgcl_example)
endif()

if (TARGET vexcl_target)
    add_amgcl_example(vexcl vexcl.cpp)
    target_link_libraries(vexcl vexcl_target)

    add_amgcl_example(solver_vexcl solver.cpp)
    target_link_libraries(solver_vexcl vexcl_target)
    target_compile_definitions(solver_vexcl PRIVATE SOLVER_BACKEND_VEXCL)

    add_amgcl_example(schur_pressure_correction_vexcl schur_pressure_correction.cpp)
    target_link_libraries(schur_pressure_correction_vexcl vexcl_target)
    target_compile_definitions(schur_pressure_correction_vexcl PRIVATE SOLVER_BACKEND_VEXCL)

    add_amgcl_example(cpr_drs_vexcl cpr_drs.cpp)
    target_link_libraries(cpr_drs_vexcl vexcl_target)
    target_compile_definitions(cpr_drs_vexcl PRIVATE SOLVER_BACKEND_VEXCL)
endif()

if (TARGET viennacl_target)
    add_amgcl_example(solver_viennacl solver.cpp)
    target_link_libraries(solver_viennacl viennacl_target)
    target_compile_definitions(solver_viennacl PRIVATE SOLVER_BACKEND_VIENNACL)

    add_amgcl_example(schur_pressure_correction_viennacl schur_pressure_correction.cpp)
    target_link_libraries(schur_pressure_correction_viennacl viennacl_target)
    target_compile_definitions(schur_pressure_correction_viennacl PRIVATE SOLVER_BACKEND_VIENNACL)

    add_amgcl_example(cpr_drs_viennacl cpr_drs.cpp)
    target_link_libraries(cpr_drs_viennacl viennacl_target)
    target_compile_definitions(cpr_drs_viennacl PRIVATE SOLVER_BACKEND_VIENNACL)
endif()

if (TARGET cuda_target)
    function(add_cuda_example example)
        configure_file(
            ${CMAKE_CURRENT_SOURCE_DIR}/${example}.cpp
            ${CMAKE_CURRENT_BINARY_DIR}/${example}.cu
            COPYONLY
            )

        cuda_add_executable(${example}_cuda ${CMAKE_CURRENT_BINARY_DIR}/${example}.cu
            OPTIONS -DSOLVER_BACKEND_CUDA)
        target_link_libraries(${example}_cuda amgcl_example cuda_target)
    endfunction()

    add_cuda_example(solver)
    add_cuda_example(schur_pressure_correction)
    add_cuda_example(cpr_drs)
endif()

add_subdirectory(mpi)
