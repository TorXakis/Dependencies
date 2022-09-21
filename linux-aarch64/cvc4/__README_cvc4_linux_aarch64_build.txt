

SHORT:
-------

current version distro: https://github.com/TorXakis/Dependencies/releases/download/cvc4_1.7/cvc4-1.7-aarch64-ubuntu-16.04.tgz

run:

  docker build -f Dockerfile.ubuntu16.04_cvc4-1.7 --target dist --output . .

this builds cvc4 static binary in docker image and outputs result in current directory 
as file  

  cvc4-1.7.tgz
  
it contains a statically linked executable z3, which you can deploy standalone.
You can extract the z3 executable with the command:

  $ tar xzvf cvc4-1.7.tgz --strip-components 2  usr/bin
  x cvc4




docs build 
===========

src https://github.com/CVC4/CVC4-archived/blob/master/INSTALL.md


Building CVC4

    ./contrib/get-antlr-3.4  # download and build ANTLR
    ./configure.sh   # use --prefix to specify a prefix (default: /usr/local)
                     # use --name=<PATH> for custom build directory
    cd <build_dir>   # default is ./build
    make             # use -jN for parallel build with N threads
    make check       # to run default set of tests
    make install     # to install into the prefix specified above



note: by default builds a dynamic linked cvc4:

    root@daf48d737b61:/# ldd /usr/local/bin/cvc4
    	linux-vdso.so.1 =>  (0x0000ffffbba31000)
    	libcvc4parser.so.7 => /usr/local/lib/libcvc4parser.so.7 (0x0000ffffbb761000)
    	libcvc4.so.7 => /usr/local/lib/libcvc4.so.7 (0x0000ffffba5cc000)
    	libgmp.so.10 => /usr/lib/aarch64-linux-gnu/libgmp.so.10 (0x0000ffffba54e000)
    	libstdc++.so.6 => /usr/lib/aarch64-linux-gnu/libstdc++.so.6 (0x0000ffffba3bf000)
    	libgcc_s.so.1 => /lib/aarch64-linux-gnu/libgcc_s.so.1 (0x0000ffffba39e000)
    	libc.so.6 => /lib/aarch64-linux-gnu/libc.so.6 (0x0000ffffba258000)
    	/lib/ld-linux-aarch64.so.1 (0x0000ffffbba06000)
    	libm.so.6 => /lib/aarch64-linux-gnu/libm.so.6 (0x0000ffffba1ab000)


dependencies
------------

 we can build cvc4 with some dependencies in it,
 for which and what configure flag to use in build,
 see https://github.com/CVC4/CVC4-archived/blob/master/INSTALL.md#optional-dependencies
 
 for an existing binary we can still see which dependencies are build-in:
 eg.
 
   for https://github.com/TorXakis/Dependencies/releases/download/cvc4_1.7/cvc4-1.7-x86_64-linux-opt
   we have:
   
             # cvc4  --show-config
             abc           : no
             cln           : yes            
             glpk          : no
             cadical       : no
             cryptominisat : no
             drat2er       : no
             gmp           : no
             lfsc          : no
             readline      : no
             symfpu        : yes
   
        
        hmmm: 
          https://github.com/CVC4/CVC4-archived/blob/master/INSTALL.md#cln--v13-class-library-for-numbers
           CLN is an alternative multiprecision arithmetic package that may offer better performance and memory footprint than GMP.
           Configure CVC4 with configure.sh --cln to build with this dependency.

           Note that CLN is covered by the GNU General Public License, version 3. If you choose to use CVC4 with CLN support, you are licensing
           CVC4 under that same license. (Usually CVC4's license is more permissive than GPL, see the file COPYING in the CVC4 source
           distribution for details.)
            => because cvc4 is included in torxakis debian package => installing gpl code!!
            
            TODO: rebuild without cln!
        
        https://github.com/CVC4/CVC4-archived/blob/master/INSTALL.md#symfpu-support-for-the-theory-of-floating-point-numbers
         SymFPU (Support for the Theory of Floating Point Numbers)

         SymFPU is an implementation of SMT-LIB/IEEE-754 floating-point operations in terms of bit-vector operations. It is required for supporting the theory of floating-point numbers and can be installed using the contrib/get-symfpu script.
         Configure CVC4 with configure.sh --symfpu to build with this dependency.
        
        https://github.com/CVC4/CVC4-archived/blob/master/INSTALL.md#cryptominisat-optional-sat-solver
         CryptoMiniSat (Optional SAT solver)

         CryptoMinisat is a SAT solver that can be used for solving bit-vector problems with eager bit-blasting. This dependency may improve
         performance. It can be installed using the contrib/get-cryptominisat script.
         
         Configure CVC4 with configure.sh --cryptominisat to build with this dependency.
        
        
    for macos build https://github.com/TorXakis/homebrew-TorXakis/blob/master/Formula/cvc4%401.7.rb
    
              args = ["--prefix=#{prefix}",
                      "--symfpu",
                      "--cryptominisat"] 

        note: just took over these options from the x64  build into the arm64 build
        
   
    Think only --symfpu  is needed for torxakis??
                      




for arM64 we needed to patch 'get-antlr-3.4'
--------------------------------------------

  * description patch

       after line
         rm -rf src/antlr3debughandlers.c && touch src/antlr3debughandlers.c

       add
          curl -Lo config.guess 'http://git.savannah.gnu.org/gitweb/?p=config.git;a=blob_plain;f=config.guess;hb=HEAD'
          curl -Lo config.sub 'http://git.savannah.gnu.org/gitweb/?p=config.git;a=blob_plain;f=config.sub;hb=HEAD'

       3 TIMES replace
         if [ "${MACHINE_TYPE}" == 'x86_64' ]; then
        with
         if [[ "${MACHINE_TYPE}" == *'64'* ]]; then

        => also matches  aarch64 !!


  * created patch file for 'get-antlr-3.4'
  
      wget https://github.com/TorXakis/Dependencies/releases/download/cvc4_1.7/cvc4-1.7_get-antlr-3.4.patch

  * apply patch
       
      patch CVC4-archived-1.8/contrib/get-antlr-3.4  cvc4-1.7_get-antlr-3.4.patch
 


build
=====

 
 
 
cvc4
-----

commands: 

    # wget https://github.com/CVC4/CVC4-archived/archive/refs/tags/1.8.tar.gz
    # tar xzvf 1.8.tar.gz
    # cd CVC4-archived-1.8/

    wget https://github.com/CVC4/CVC4-archived/archive/refs/tags/1.7.tar.gz
    tar xzvf 1.7.tar.gz
    cd CVC4-archived-1.7/
    
    #cp ../get-antlr-3.4 ./contrib/get-antlr-3.4
    #./contrib/get-antlr-3.4 # patch version

    # get patch
    wget https://github.com/TorXakis/Dependencies/releases/download/cvc4_1.7/cvc4-1.7_get-antlr-3.4.patch

    #apply patch
    patch CVC4-archived-1.8/contrib/get-antlr-3.4  cvc4-1.7_get-antlr-3.4.patch

    

    ./configure.sh --static-binary
    cd build 
    make 

  
note:
  make -j 8  #  builds faster!
  


static linking 
----------------

  
configure.sh --help
  ..
  --static                 build static libraries and binaries [default=no]
  --static-binary          enable/disable static binaries                      
  ..

  => after inspection: configure.sh  and CMakeLists.txt 
  
           the 'static' and 'static-binary' trigger the same settings in cmake
          
                shared=OFF; static_binary=ON  
          
          instead the default of 
          
                shared=ON; static_binary=OFF


=> note:  DON't need static compiling on mac!!
 
 

static building fails on macos
------------------------------
./configure.sh --static
cd build 
make 

# get error at linking
#  [100%] Linking CXX executable ../../bin/cvc4
#  ld: library not found for -lcrt0.o
#  clang: error: linker command failed with exit code 1 (use -v to see invocation)

https://lists.sr.ht/~sircmpwn/public-inbox/%3C3A12328E-5EC6-4E34-A676-64A3039A2AF7%40herrbischoff.com%3E
  
  Try removing `-static` from LDFLAGS in the Makefile. The reason being is 
  that on macOS, there isn't a static version of the crt0.o library.

  See 
  https://stackoverflow.com/questions/3801011/ld-library-not-found-for-lcrt0-o-on-osx-10-6-with-gcc-clang-static-flag 
  for details.

=> cannot static link on macos

 
 
 
old: on older macos needed --python3 flag
------------------------------------------
# on m1 mac for cvc4 build (only for build):
   => complaints about missing toml python library
      had to fix it in system python2

          sudo python2 -m pip uninstall  toml
          python2 -m pip install --user  toml
          # note: something wrong with toml installed on the system dir, so instead install it in user packages dir
          
  => however LATER I got an update which remove python2 from my mac
     and then it worked also just fine with python3
     
     I found out, that by default it does
     
     from CMakeLists.txt
     

      if(USE_PYTHON2)
        find_package(PythonInterp 2.7 REQUIRED)
      elseif(USE_PYTHON3)
        find_package(PythonInterp 3 REQUIRED)
      else()
        find_package(PythonInterp REQUIRED)
      endif()
  
    from configure.sh
    
       --python2                prefer using Python 2 (also for Python bindings)
       --python3                prefer using Python 3 (also for Python bindings)
       --python-bindings        build Python bindings based on new C++ API
       
     => so with   ' --python3'  we can always enforce the use of python3 in the build (only for the build, not for runtime!)
        note this option does not enable the python bindings. For that you must also give the option   --python-bindings 
     