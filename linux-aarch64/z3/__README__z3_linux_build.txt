SHORT:
-------

current version distro: https://github.com/TorXakis/Dependencies/releases/download/z3-4.8.7/z3-4.8.7-aarch64-ubuntu-16.04.tgz


run:

  docker build -f Dockerfile.ubuntu16.04_z3-4.8.7 --target dist --output . .

this builds z3 static binary in docker image and outputs result in current directory 
as file  

  z3-4.8.7.tgz
  
it contains a statically linked executable z3, which you can deploy standalone.
You can extract the z3 executable with the command:

  $ tar xzvf z3-4.8.7.tgz --strip-components 2  usr/bin
  x z3



====
 z3
====


DOCUMENTATION:
--------------

from https://github.com/Z3Prover/z3 (README.md) how to build/install with gcc:

    python scripts/mk_make.py
    cd build
    make
    sudo make install

options of configuring the build:

    $ python3 scripts/mk_make.py --help
    opt = --help, arg =
    mk_make.py: Z3 Makefile generator

    This script generates the Makefile for the Z3 theorem prover.
    It must be executed from the Z3 root directory.

    Options:
      -h, --help                    display this message.
      -s, --silent                  do not print verbose messages.
      -p <dir>, --prefix=<dir>      installation prefix (default: /usr).
      --pypkgdir=<dir>              Force a particular Python package directory (default /usr/lib/python3/dist-packages)
      -b <subdir>, --build=<subdir>  subdirectory where Z3 will be built (default: build).
      --githash=hash                include the given hash in the binaries.
      --git-describe                include the output of 'git describe' in the version information.
      -d, --debug                   compile Z3 in debug mode.
      -t, --trace                   enable tracing in release mode.
      --x86                         force 32-bit x86 build on x64 systems.
      -m, --makefiles               generate only makefiles.
      --dotnet                      generate .NET platform bindings.
      --dotnet-key=<file>           sign the .NET assembly using the private key in <file>.
      --java                        generate Java bindings.
      --ml                          generate OCaml bindings.
      --js                          generate JScript bindings.
      --python                      generate Python bindings.
      --staticlib                   build Z3 static library.
      --staticbin                   build a statically linked Z3 binary.
      -g, --gmp                     use GMP.
      --gprof                       enable gprof
      --log-sync                    synchronize access to API log files to enable multi-thread API logging.
      --single-threaded             non-thread-safe build

    Some influential environment variables:
      CXX        C++ compiler
      CC         C compiler
      LDFLAGS    Linker flags, e.g., -L<lib dir> if you have libraries in a non-standard directory
      CPPFLAGS   Preprocessor flags, e.g., -I<include dir> if you have header files in a non-standard directory
      CXXFLAGS   C++ compiler flags
      JDK_HOME   JDK installation directory (only relevant if -j or --java option is provided)
      JNI_HOME   JNI bindings directory (only relevant if -j or --java option is provided)
      OCAMLC     Ocaml byte-code compiler (only relevant with --ml)
      OCAMLFIND  Ocaml find tool (only relevant with --ml)
      OCAMLOPT   Ocaml native compiler (only relevant with --ml)
      OCAML_LIB  Ocaml library directory (only relevant with --ml)
      CSC        C# Compiler (only relevant if .NET bindings are enabled)
      GACUTIL    GAC Utility (only relevant if .NET bindings are enabled)
      Z3_INSTALL_BIN_DIR Install directory for binaries relative to install prefix
      Z3_INSTALL_LIB_DIR Install directory for libraries relative to install prefix
      Z3_INSTALL_INCLUDE_DIR Install directory for header files relative to install prefix
      Z3_INSTALL_PKGCONFIG_DIR Install directory for pkgconfig files relative to install prefix


build fixed version for z3-4.8.7
---------------------------------

    apt-get update
    apt-get install -y wget python3 binutils gcc g++ make


    wget  https://github.com/Z3Prover/z3/archive/z3-4.8.7.tar.gz
    tar xzvf z3-4.8.7.tar.gz
    cd z3-z3-4.8.7

    PREFIX=/tmp/z3-4.8.7
    python3 scripts/mk_make.py --prefix=$PREFIX  --staticbin
    cd build 
    make -j$(nproc)
    make install 
    tar -C /tmp/ -czvf z3-4.8.7.tgz z3-4.8.7


build for specific platform
----------------------------

x64
 => just do build on x64 machine

arm64
 => just do build on a arm64 machine
    eg. build in linux docker image on macbook with m1 processor (arm64) 
    

static vs dynamic build
-----------------------

Run your formula’s test using brew test <formula>. The test should finish without any errors.


without --staticbin

    # du -h /tmp/z3-4.8.7/bin/z3
    23M	/tmp/z3-4.8.7/bin/z3
    

    # ldd /usr/local/bin/z3
    	linux-vdso.so.1 =>  (0x0000ffffbda07000)
    	libpthread.so.0 => /lib/aarch64-linux-gnu/libpthread.so.0 (0x0000ffffbd9b0000)
    	libstdc++.so.6 => /usr/lib/aarch64-linux-gnu/libstdc++.so.6 (0x0000ffffbd821000)
    	libm.so.6 => /lib/aarch64-linux-gnu/libm.so.6 (0x0000ffffbd774000)
    	libgcc_s.so.1 => /lib/aarch64-linux-gnu/libgcc_s.so.1 (0x0000ffffbd753000)
    	libc.so.6 => /lib/aarch64-linux-gnu/libc.so.6 (0x0000ffffbd60d000)
    	/lib/ld-linux-aarch64.so.1 (0x0000ffffbd9dc000)

with --staticbin    
    
    # file z3
    z3: ELF 64-bit LSB executable, ARM aarch64, version 1 (GNU/Linux), statically linked, for GNU/Linux 3.7.0, BuildID[sha1]=11b4469ae412fbe4d636e3a126be956c0a5d117e, not stripped
    # ldd z3
    	not a dynamic executable
    # du -sh z3
    26M	z3
    
advantage of static binary: only need to install /usr/local/bin/z3

    extract z3 binary from distribution package:
        $ tar xzvf z3-4.8.7_aarch64_ubuntu16.04.uses_shared_libs.tgz --strip-components 3  usr/local/bin
        x z3

    example: install on ubuntu 16.04 docker image
      $ docker cp z3 ubuntu_16.04:/usr/local/bin
      # z3 works!


Dockerfile to build z3
----------------------

Important:
  note that when running this docker build on a x64 machine,
  you get x64 build of z3, and when building on a arm64 machine you get 
  an arm64 build!

note: in Dockerfile we use as prefix /usr instead of /usr/local because
      we are building a generic linux package which are always installed in /usr/ 

run:

  docker build -f Dockerfile.ubuntu16.04_z3-4.8.7 --target dist --output . .

this builds z3 static binary in docker image and outputs result in current directory 
as file  

  z3-4.8.7.tgz
  
it contains a statically linked executable z3, which you can deploy standalone.
You can extract the z3 executable with the command:

  $ tar xzvf z3-4.8.7.tgz --strip-components 2  usr/bin
  x z3

  
inspect it: 
  
  $ du -sh z3
  26M	z3
  $ file z3
  z3: ELF 64-bit LSB executable, ARM aarch64, version 1 (GNU/Linux), statically linked, for GNU/Linux 3.7.0, BuildID[sha1]=11b4469ae412fbe4d636e3a126be956c0a5d117e, with debug_info, not stripped
                                       `-> note: we build in docker on m1 macbook
