
SHORT:
-------

current version: https://github.com/TorXakis/Dependencies/releases/download/z3-4.8.7/linuxdeploy-aarch64.v20220914.05855fa.AppImage

########################################################################################################################
#                      USAGE
########################################################################################################################
# run below command on arm64 machine(eg. m1 macbook) and the build distribution file will be put in the ./dist/ folder
#
#    docker build -f Dockerfile_linuxdeploy.aarch64.v20220914.05855fa --target dist --output dist/ .
#
# Note: the resulting linuxdeploy.AppImage works on ubuntu16.04(verified) and newer !! 
#       Although linuxdeploy is build on 20.04 it builds static binaries which work on older linux versions.   
########################################################################################################################




============
linuxdeploy
============

IMPORTANT: https://docs.appimage.org/introduction/concepts.html   


How linuxdeploy constructed?
----------------------------
   
   linuxdeploy
    embeds plugin 
      linuxdeploy-plugin-appimage (for --output option to build AppImage with appimagetool)
        which embeds 
         appimagetool (builds AppImage from AppDir)

Does linuxdeploy work on older linux systems?
---------------------------------------------
         
  linuxdeploy and    linuxdeploy-plugin-appimage github repos 
     only create binaries which are static linked 
     => no dependency on dynamic libs and  only depends on system calls
       because  only depends on system calls is good portable to older systems!
    => both have therefore only a continuous release. (no older releases!)
       which is build on ubuntu latest (in github actions workflow)
       
  however  appimagetool has content which is dependent on several dynamic libraries
        => dependency on dyn. libs makes it less portable to older systems!
        => has therefore also older releases available!
        => therefore take oldest release which has arm64 build 
        
         
        
    
  IMPORTANT: https://github.com/linuxdeploy/linuxdeploy-plugin-appimage/blob/master/ci/build-appimage.sh
           always takes latest appimagetool build 
           => so can give problem on older linux systems!! 
              in that case don't use the linuxdeploy-plugin-appimage plugin in linuxdeploy ( --output  option uses this plugin)
                                          `-> see https://docs.appimage.org/introduction/software-overview.html#linuxdeploy
                                               https://docs.appimage.org/packaging-guide/from-source/native-binaries.html#bundling-resources-into-the-appdir
                                               https://docs.appimage.org/packaging-guide/from-source/native-binaries.html#build-appimages-from-appdir-using-linuxdeploy
              but instead only use linuxdeploy to setup the AppDir (don't use --output)
                                         `-> https://docs.appimage.org/packaging-guide/from-source/native-binaries.html#bundling-resources-into-the-appdir
              and use a separate older appimagetool (as AppImage)  to make from AppDir/ and AppImage
                                        -> see
                                           https://docs.appimage.org/introduction/software-overview.html#appimagetool
                                           https://docs.appimage.org/packaging-guide/manual.html?highlight=appimagetool#creating-an-appimage-from-the-appdir
                                        
    
  => In our build of linuxdeploy to let appimagetool work on ubuntu 16.04 we use release 12 (oldest one having arm64 build!)
     then we can use our linuxdeploy with the buildin appimagetool on ubuntu16.04



different versions builds
==========================

note: we can use 20.04 build on older systems like ubuntu 16.04 -> see  "Does linuxdeploy work on older linux systems?" above for explanation.

16.04 Dockerfile NOT finished because:  

       $ docker buildx build -t arm64_linuxdeploy_16.04 .    
        -> ERROR: CMake 3.6 or higher is required.  You are running version 3.5.1
    
    tried to install newer version of cmake from different package repository
    first tried older linuxdeploy version -> maybe needs older version cmake
    then tried installing newer cmake
    however later discovered that that repo only has arm64 builds of cmake for ubuntu 20.04 but not for ubuntu 18.04 and 16.04
      from https://apt.kitware.com
       The 16.04 and 18.04 repositories support x86 (32-bit and 64-bit), 
       and the 20.04 repository supports x86 (32-bit and 64-bit) and ARM (32-bit and 64-bit).
       
     => so could not build on 16.04 
       (unless maybe when take even an older version of linuxdeploy then 8f59b3ef4abe34b6c5074c5f8502c4dea5e6a01e)

18.04  Dockerfile works, but only builds binary (NO appimage for linuxdeploy with plugins)     
     =>  however could build on 18.04 because the cmake version coming with ubuntu 16.04 is good enough for
     => we use linuxdeploy version 8f59b3ef4abe34b6c5074c5f8502c4dea5e6a01e 
        because problems with boost library on newest version  (probably because we need newer c++/boost verion )
       NOTE: build works without needing -DBUILD_TESTING=OFF   
       NOTE: newest version has custom shells scripts to run the cmake builds,
             but these aren't not yet there in the older version => just use cmake there directly
      
         
20.04 Dockerfile  => full build 
    => because newest linuxdeploy code didn't work on 18.04 we did try it on 20.04
       and this build works,
       however we need to disable building test code with cmake option -DBUILD_TESTING=OFF 
       because without this option the testcode gives a not finding threads library?? -> didn't have time to really solve this.
       
   => full build with linuxdeploy appimage with appimagetool plugin integrated
      which uses the custom shells scripts to run the cmake builds
   => note for   appimagetool plugin  used older build of  appimagetool so that still works on ubuntu 16.04 
     see "Does linuxdeploy work on older linux systems?" above for explanation.

  
Note prefer newest source code of linuxdeploy, because probably is better.

====================
 cmake  tips
====================

about cmake options
====================

when having problems with cmake build you could  play with the options in the build.

Question: How do we find which cmake options there are?
Answer:


    https://stackoverflow.com/questions/16851084/how-to-list-all-cmake-build-options-and-their-default-values

    * cmake -LA
 
 
     You may also be interested in adding -H to show more help information about each option as previously mentioned at:
      https://stackoverflow.com/a/53075317/895245

         cmake -LAH           => BEST!!
 
 
    * ccmake ncurses            => convenient

         apt-get install cmake-curses-gui
         ccmake ..
 
 
So easiest solution is with 'ccmake /linuxdeploy/'  (curses-gui) cmdline app we can easily turn off some option...
       `->  apt-get install -y cmake-curses-gui


example in 20.04/Dockerfile 
----------------------------
we had problem with a build, so we look at available options

  ccmake shows options :
           BUILD_GMOCK                      ON
           BUILD_TESTING                    ON
           CCACHE                           CCACHE-NOTFOUND
           CIMG_H_DIR                       /usr/include
           CImg_INCLUDE_DIR                 /usr/include
           CMAKE_BUILD_TYPE
           CMAKE_INSTALL_PREFIX             /usr/local
           ENABLE_COVERAGE                  OFF
           FETCHCONTENT_BASE_DIR            /build/build/_deps
           FETCHCONTENT_FULLY_DISCONNECTE   OFF
           FETCHCONTENT_QUIET               ON
           FETCHCONTENT_SOURCE_DIR___LD_G
           FETCHCONTENT_UPDATES_DISCONNEC   OFF
           FETCHCONTENT_UPDATES_DISCONNEC   OFF
           INSTALL_GTEST                    ON
           USE_CCACHE                       ON


so we tried adding options to cmake command:
  -DCMAKE_INSTALL_PREFIX=/usr -DBUILD_GMOCK=OFF -DBUILD_TESTING=OFF  -DUSE_CCACHE=OFF -DINSTALL_GTEST=OFF
and this worked, we could narrow it down, and also only
  -DINSTALL_GTEST=OFF
works!!   seems  only testing code needs newer version of cmake

      
search terms in CMakeLists.txt files in project
===============================================

get "find_package" in cmake
---------------------------


$ searchterm=find_package; for f in `find . | grep CMakeLists.txt`; do   if grep -i $searchterm $f &>/dev/null; then  echo $f;    grep -i $searchterm $f;    echo; fi ; done

./src/CMakeLists.txt
find_package(Boost REQUIRED COMPONENTS filesystem regex)

./src/core/CMakeLists.txt
find_package(Threads)
find_package(CImg)
    find_package(CImg REQUIRED)

get "option" in cmake
---------------------
    
    
$ searchterm=option; for f in `find . | grep CMakeLists.txt`; do   if grep -i $searchterm $f &>/dev/null; then  echo $f;    grep -i $searchterm $f;    echo; fi ; done
./_deps/__ld_gtest-src/googletest/CMakeLists.txt
# For more options, run 'ctest --help'.
option(
option(gtest_build_tests "Build all of gtest's own tests." OFF)
option(gtest_build_samples "Build gtest's sample programs." OFF)
option(gtest_disable_pthreads "Disable uses of pthreads in gtest." OFF)
option(
include(cmake/hermetic_build.cmake OPTIONAL)
  option(BUILD_SHARED_LIBS "Build shared libraries (DLLs)." OFF)
# gtest_build_samples option to ON.  You can do it by running ccmake
# gtest_build_tests option to ON.  You can do it by running ccmake
  cxx_test(googletest-options-test gtest_main)

./_deps/__ld_gtest-src/googlemock/CMakeLists.txt
# For more options, run 'ctest --help'.
option(gmock_build_tests "Build all of Google Mock's own tests." OFF)
include("${gtest_dir}/cmake/hermetic_build.cmake" OPTIONAL)
  option(BUILD_SHARED_LIBS "Build shared libraries (DLLs)." OFF)
# gmock_build_tests option to ON.  You can do it by running ccmake
      add_compile_options("-Wa,-mbig-obj")

./_deps/__ld_gtest-src/CMakeLists.txt
include(CMakeDependentOption)
option(BUILD_GMOCK "Builds the googlemock subproject" ON)
option(INSTALL_GTEST "Enable installation of googletest. (Projects embedding googletest may want to turn this OFF.)" ON)

./lib/args/CMakeLists.txt
option(ARGS_BUILD_EXAMPLE "Build example" ON)
option(ARGS_BUILD_UNITTESTS "Build unittests" ON)
        target_compile_options(argstest PRIVATE /W4 /WX /bigobj)
        target_compile_options(argstest PRIVATE -Wall -Wextra -Werror -pedantic -Wshadow -Wunused-parameter)

./src/CMakeLists.txt
    target_link_options(linuxdeploy PUBLIC -static -static-libgcc -static-libstdc++)

./tests/simple_executable/CMakeLists.txt
target_link_options(simple_executable_static PUBLIC -static -static-libgcc -static-libstdc++)



                  
