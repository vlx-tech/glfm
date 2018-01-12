if (FIPS_IOS OR FIPS_ANDROID)

    fips_begin_lib(glfm)

        include_directories(../include)

        set(GLFM_HEADERS ../include/glfm.h)
        
        if (FIPS_ANDROID)
            set(GLFM_SRC ../src/glfm_platform.h ../src/glfm_platform_android.c ${ANDROID_NDK_ROOT}/sources/android/native_app_glue/android_native_app_glue.c)

            # Set NDEBUG for android_native_app_glue to remove some superfluous logging
            set_source_files_properties(${ANDROID_NDK_ROOT}/sources/android/native_app_glue/android_native_app_glue.c PROPERTIES COMPILE_FLAGS "-DNDEBUG")
            fips_libs(log android EGL GLESv2)
        else()
            set(GLFM_SRC ../src/glfm_platform.h ../src/glfm_platform_ios.m)
            set(CMAKE_OSX_SYSROOT "iphoneos")
            set(GLFM_COMPILE_FLAGS "-Wno-objc-interface-ivars -Wno-objc-missing-property-synthesis -Wno-direct-ivar-access")
            fips_frameworks_osx(Foundation CoreGraphics UIKit OpenGLES QuartzCore)
        endif()

        fips_files(${GLFM_SRC} ${GLFM_HEADERS})

    #@@@
    #if (CMAKE_SYSTEM_NAME MATCHES "Android")
    #    find_library(log-lib log)
    #    find_library(android-lib android)
    #    find_library(EGL-lib EGL)
    #    find_library(GLESv2-lib GLESv2)
    #    target_link_libraries(glfm ${log-lib} ${android-lib} ${EGL-lib} ${GLESv2-lib})
    
    fips_end_lib()

    if (FIPS_ANDROID)
        target_include_directories(glfm PRIVATE ${ANDROID_NDK_ROOT}/sources/android/native_app_glue/)
    endif()
    
    # silence some warnings
    if (FIPS_CLANG)
        set_target_properties(glfm PROPERTIES COMPILE_FLAGS "-Weverything -Wwrite-strings -Wno-padded -Wno-covered-switch-default ${GLFM_COMPILE_FLAGS}")    
    elseif (FIPS_GCC)
        set_target_properties(glfm PROPERTIES COMPILE_FLAGS "-Wall -Wextra -Wwrite-strings ${GLFM_COMPILE_FLAGS}")
    elseif (FIPS_MSVC)
        set_target_properties(glfm PROPERTIES COMPILE_FLAGS "/Wall ${GLFM_COMPILE_FLAGS}")
    endif()

endif()