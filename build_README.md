# Windows - Release mode
- Before running test install fonts from oiio-kaarrot/src/fonts to C:/Windows/Fonts

  - drag and drop does not work !!!
    instead
  cp C:/Users/kuba/SRC/OIIO/oiio-kaarrot/src/fonts/Droid_Sans/DroidSans.ttf c:/Windows/Fonts
  cp C:/Users/kuba/SRC/OIIO/oiio-kaarrot/src/fonts/Droid_Sans/DroidSans-Bold.ttf  C:/Windows/Fonts
  cp C:/Users/kuba/SRC/OIIO/oiio-kaarrot/src/fonts/Droid_Sans_Mono/DroidSansMono.ttf C:/Windows/Fonts/
  cp C:/Users/kuba/SRC/OIIO/oiio-kaarrot/src/fonts/Droid_Serif/DroidSerif.ttf  C:/Windows/Fonts/
  cp C:/Users/kuba/SRC/OIIO/oiio-kaarrot/src/fonts/Droid_Serif/DroidSerif-Bold.ttf  C:/Windows/Fonts/
  cp C:/Users/kuba/SRC/OIIO/oiio-kaarrot/src/fonts/Droid_Serif/DroidSerif-BoldItalic.ttf  C:/Windows/Fonts/
  cp C:/Users/kuba/SRC/OIIO/oiio-kaarrot/src/fonts/Droid_Serif/DroidSerif-Italic.ttf  C:/Windows/Fonts/
   - on windows there is not automated way to install DroidSans font (which is used in oiiotool test)
   - the 'cour' is found, test don't hard error but vaey slighlty due to a differnet font used which is hard to figure out and determine which font is requires/missing to make the test pass


- set up Python with Numpy
```
python3 -m venv C:/Users/kuba/SRC/OIIO/venv/oiio 
C:/Users/kuba/SRC/OIIO/venv/oiio/Scripts/Activate.ps1
python -m pip install numpy==1.26.4
```

```
cmake .. -DOIIO_DOWNLOAD_MISSING_TESTDATA=ON -DOpenImageIO_BUILD_MISSING_DEPS=all -DINSTALL_FONTS=ON -DCMAKE_GENERATOR_PLATFORM=x64 -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=install_prefix
cmake --build . --config=Release
cmake --install . --config=Release
```

```
$env:OIIO_LOAD_DLLS_FROM_PATH=1
$env:PATH="C:/Users/kuba/SRC/OIIO/oiio-kaarrot/__build/bin/Release;C:/Users/kuba/SRC/OIIO/oiio-kaarrot/__build/deps/dist/bin;$env:PATH"
$env:PYTHONPATH="C:/Users/kuba/SRC/OIIO/oiio-kaarrot/__build/bin/Release"

- in order to loaod the dll whcih depends on other dlls

import os
os.add_dll_directory('C:/Users/kuba/SRC/OIIO/oiio-kaarrot/__build/bin/Release')
os.add_dll_directory('C:/Users/kuba/SRC/OIIO/oiio-kaarrot/__build/deps/dist/bin')
import OpenImageIO

## This also work (runs __init__.py which tokenize the PATH)
- Note!!! With this option we need to install with -DCMAMKE_INSTALL_PREFIX=./install_prefix

$env:OIIO_LOAD_DLLS_FROM_PATH=1
$env:PYTHONPATH='C:/Users/kuba/SRC/OIIO/oiio-kaarrot/__build/install_prefix/lib/python3.11/site-packages/'
import OpenImageIO.PyOpenImageIO_kuba.OpenImageIO_kuba

## Another approach - hopefully will work with Tests
- in bin/Release create folder OpenImageIO
- move OpenImageIO.cp311-win_amd64 and __inti__.py which import fomr .OpenImageIO
- set  $env:PYTHONPATH="C:/Users/kuba/SRC/OIIO/oiio-kaarrot/__build/bin/Release"


```

!!!! PATH is no longer used to resolveing libraries 
https://stackoverflow.com/questions/20201868/importerror-dll-load-failed-the-specified-module-could-not-be-found#:~:text=Since%20Python%203.8%2C%20it%20is,resolving%20DLLs%20of%20binary%20modules!


### Testing what dependencies we need
dumpbin /dependents OpenImageIO.cp311-win_amd64.pyd


# THIS DOES NOT WORK - apparently if the build folder is too long OpenColorIO failes to configure (windows)
- changing the directory from cmake-build-release-visual-studio to build_clion !!!



# THis doesn not work when building on windows OpenColorIO
uting command: C:/Users/kuba/scoop/shims/cmake.EXE -DCMAKE_EXPORT_COMPILE_COMMANDS:BOOL=TRUE -DOIIO_DOWNLOAD_MISSING_TESTDATA=ON -DOpenImageIO_BUILD_MISSING_DEPS=all -DINSTALL_FONTS=ON --no-warn-unused-cli -SC:/Users/kuba/SRC/OIIO/oiio-kaarrot -Bc:/Users/kuba/SRC/OIIO/oiio-kaarrot/__build_vscode_Release_MSVC -G "Visual Studio 17 2022" -T host=x64 -A x64


ctest -C Release -E broken --force-new-ctest-process --output-on-failure

### Testing oiiotools
cmake --build . --config=Release
cmake --install . --config=Release
bin/Release/oiiotool  --pattern constant:color=0.5,0.0,0.0 64x64 3 --text A -attrib oiio:subimagename layerA --pattern constant:color=0.0,0.5,0.0 64x64 3 --text B -attrib oiio:subimagename layerB --pattern constant:color=0.0,0.0,0.5 64x64 3 --text C -attrib oiio:subimagename layerC --pattern constant:color=0.5,0.5,0.0 64x64 3 --text D -attrib oiio:subimagename layerD --siappendall -d half -o subimages-4.exr

ctest -C Release -E broken --force-new-ctest-process --output-on-failure -R oiiotool-cop



```

## Notes
- note when building in Debug mode to install python311_d.lib (TODO: check how to automate debug build)
- while building from VSCode after initial failed configuration edit CMakeCache.txt - append 'all'
  OpenImageIO_BUILD_MISSING_DEPS:STRING=all
- current building as RelWithDebInfo fails (TODO: it fails early but not sure why )


# Docker 

Check: `C:/Users/kuba/SRC/OIIO/docker_kuba/Dockerfile`

# Linux build (Summary) - Old
- makefile (recommended) from the root director
```
CMAKE_BUILD_PARALLEL_LEVEL=14 make OpenImageIO_BUILD_MISSING_DEPS=all

CMAKE_BUILD_PARALLEL_LEVEL=14 make install

CMAKE_PREFIX_PATH=$(pwd)/dist/ PYTHONPATH=$(pwd)/lib/python/site-packages make test
```

```
cmake .. -DOpenImageIO_BUILD_MISSING_DEPS=all
cmake .. --fresh
make install
```

- WSL linux accessing build on Window is SLOOOOW

# Setup ENV - runtime
```
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$HOME/SRC/OIIO/oiio-kaarrot/dist/lib
export PYTHONPATH=$PYTHONPATH:$HOME/SRC/OIIO/oiio-kaarrot/dist/lib/python3.10/site-packages 
```




######################################################################
# Things to remember - Linux Build

- run `cmake ..` again, after initial configuration: `cmake .. -DOpenImageIO_BUILD_MISSING_DEPS=all`
Tests
- install fonts
- set python3 executable: `export Python_EXECUTABLE=python3`
- set PYTHONPATH 
- set CMAKE_PREFIX_PATH

Looks like the change to include is not required, it is more an issue with cmake not picking up the build paths
# Force rescan it:
`cmake .. --fresh`


# Running test with cmake  <-------------------- current

```
rm -rf ../dist  (or run make realclean)
rm -rf *
cmake .. -DOpenImageIO_BUILD_MISSING_DEPS=all
cmake ..
make -j 14
make install 
```

# copy fonts
`sudo cp -p ~/SRC/OIIO/OpenImageIO/dist/share/fonts/OpenImageIO/* /usr/local/share/fonts/`


# Download test images (before configure) as some test are going to fail otherwisze

```
The following tests FAILED:
          5 - igrep (Failed)
         41 - texture-levels-stochaniso (Failed)
         42 - texture-levels-stochmip (Failed)
         76 - texture-levels-stochaniso.batch (Failed)
         77 - texture-levels-stochmip.batch (Failed)
        123 - unit_timer (Failed)
```
# Download external image repositories into parent folder

```
cd ..

wget https://github.com/AcademySoftwareFoundation/OpenImageIO-images/archive/refs/heads/master.zip
mv master.zip oiio-images.zip
unzip oiio-images.zip
mv OpenImageIO-images-master oiio-images

wget https://github.com/AcademySoftwareFoundation/openexr-images/archive/refs/heads/main.zip
mv main.zip openexr-images.zip
unzip openexr-images.zip
mv openexr-images-main/ openexr-images

wget --recursive --no-parent https://www.cv.nrao.edu/fits/data/tests/
mv www.cv.nrao.edu/fits/data/tests/ fits-images
```

- with that we should have all the tests passing (except timer one):
```
99% tests passed, 1 tests failed out of 191

Total Test time (real) = 796.10 sec

The following tests FAILED:
        177 - unit_timer (Failed)
```

# Set python3 executable as python may not be available  

export Python_EXECUTABLE=python3

# Test

`cd build`
`make install` - make sure to copy CMake files
`CMAKE_PREFIX_PATH=$(pwd)/../dist/ PYTHONPATH=$(pwd)/lib/python/site-packages ctest -E broken --force-new-ctest-process --output-on-failure`

- Run single test:
`CMAKE_PREFIX_PATH=$(pwd)/../dist/ ctest -R cmake-consumer -V`

- Run single test Python:
`CMAKE_PREFIX_PATH=$(pwd)/../dist/ PYTHONPATH=$(pwd)/lib/python/site-packages ctest -R python-imagebuf`



# Makefile
## Build
```
make config MY_CMAKE_FLAGS="-DOpenImageIO_BUILD_MISSING_DEPS=all"
pushd build
cmake ..
popd

CMAKE_BUILD_PARALLEL_LEVEL=14 make build

make install
```

## Test

```
export Python_EXECUTABLE=python3
export CMAKE_PREFIX_PATH=$(pwd)/dist/
export PYTHONPATH=$(pwd)/lib/python/site-packages:$PYTHONPATH
make test
make test TEST=docs-examples-cpp

```

# VScode
The advantage of configureing from IDE is out of the box 
- Cmake delete cache and configure
- Edit cache, set `OpenImageIO_BUILD_MISSING_DEPS:STRING=all`
- CMake configure - to rebuild
- CMake configure again - to refresh cmake
- Build
- ninja install



# Python

////////////////////  Python

```
cd __build_vscode_Debug_Clang
PYTHONPATH=$(pwd)/lib/python/site-packages LD_LIBRARY_PATH=$(pwd)/lib python3

import OpenImageIO as oiio

spec = oiio.ImageSpec (128, 64, 3, "float")
out = oiio.ImageOutput.create ("aaa.exr")
out.open ("multipart.exr", (spec,))
img = oiio.ImageBufAlgo.fill ((0.5, 0.0, 0.0), spec.roi)
img.write (out)
out.close()

b = oiio.ImageBuf (oiio.ImageSpec(320,240,4,oiio.UINT16))

b = oiio.ImageBuf ("../common/textures/grid.tx")
b.read()
```


```
gdb --args python3 ~/temp/aaa.py

import OpenImageIO as oiio

spec = oiio.ImageSpec (128, 64, 3, "float")
out = oiio.ImageOutput.create ("aaa.exr")
out.open ("multipart.exr", (spec,))
img = oiio.ImageBufAlgo.fill ((0.5, 0.0, 0.0), spec.roi)
img.write (out)
out.close()
```


///////////////




(WSL) 99% tests passed, 1 tests failed out of 191

Total Test time (real) = 763.28 sec

The following tests FAILED:
        177 - unit_timer (Failed)





# Build on windows with vcpk / bash   - similar error due to missing font, also was not able to figure out Python not loading

 - desc: windows-2022
            runner: windows-2022
            vsver: 2022
            generator: "Visual Studio 17 2022"
            # openexr_ver: v3.2.4
            python_ver: "3.9"
            # simd: sse4.2
            # setenvs: export OpenImageIO_BUILD_MISSING_DEPS=none
   env:
      PYTHON_VERSION: ${{matrix.python_ver}}
      CMAKE_GENERATOR: ${{matrix.generator}}
      OPENEXR_VERSION: ${{matrix.openexr_ver}}
      USE_SIMD: ${{matrix.simd}}

# export PYTHON_VERSION=3.9
```
export GITHUB_ENV=aaa_kuba.txt
export PYTHON_VERSION=3.11
export CMAKE_GENERATOR="Visual Studio 17 2022"
export CMAKE_CXX_STANDARD=17
export OPENEXR_VERSION=v3.2.4
export USE_SIMD=0

```

install vspkg 
- maybe also moddify vscpk direcotry to c:/vcpkg
- currently lives unders scoop
export GITHUB_ENV=aaa_kuba.txt  <---- remove the file when start over

also remmove build, ext in the project root 

```
src/build-scripts/ci-startup.bash
```

- modify path to before runing gh-win-installdeps.bash:
DEP_DIR="$PWD/ext/dist"
VCPKG_INSTALLATION_ROOT=C:/Users/kuba/scoop/apps/vcpkg/current
- update also python to use version 3.11 (from Scoop)

- before you run as this for some reson at the moment is not set for me (missing env_varaible). You can set these as zlib in vcpk is not discovered

```
export DEP_DIR="$PWD/ext/dist"
export VCPKG_INSTALLATION_ROOT=C:/Users/kuba/scoop/apps/vcpkg/current
export CMAKE_PREFIX_PATH="$CMAKE_PREFIX_PATH;$DEP_DIR"
export CMAKE_PREFIX_PATH="$CMAKE_PREFIX_PATH;$VCPKG_INSTALLATION_ROOT/installed/x64-windows-release"
export PATH="$PATH:$DEP_DIR/bin:$DEP_DIR/lib:$VCPKG_INSTALLATION_ROOT/installed/x64-windows-release/bin:/bin:$PWD/ext/dist/bin:$PWD/ext/dist/lib"

```
- also add support for python 3.11
```
elif [[ "$PYTHON_VERSION" == "3.11" ]] ; then
    export CMAKE_PREFIX_PATH="$CMAKE_PREFIX_PATH;/c/Users/kuba/scoop/apps/python/3.11.0"
    export Python_EXECUTABLE="/c/Users/kuba/scoop/apps/python/3.11.0/python.exe"
    export PYTHONPATH=$OpenImageIO_ROOT/lib/python${PYTHON_VERSION}/site-packages
```

```
src/build-scripts/gh-win-installdeps.bash
```

```
src/build-scripts/ci-build.bash
```
- still errors for missing JML, OpenCOlorIO, adding for now -DOpenImageIO_BUILD_MISSING_DEPS=all in ci-build.sh
- add this completes

- add -p to `mkdir -p ${OIIO_BUILD_DIR}/cmake-save || true`
- need to install

```
BUILDTARGET=install CMAKE_BUILD_TYPE=Release src/build-scripts/ci-build.bash
```

export OpenImageIO_ROOT=/c/Users/kuba/SRC/OIIO/oiio-kaarrot/build

src/build-scripts/ci-test.bash



## What ares have we coverd so far while builind libpng branch on PowerShell

- Make sure all dependneies build locally
- set correct PATH for oiiotool to work 
- fix pythonpath by copying .pyd and __init__ file into OpenImageIO subfolder.
  set required varaible in the PowerShell: $env:OIIO_LOAD_DLLS_FROM_PATH=1
  This was neccessary to setup dynamic library paths from PATH so that we can load
- Install all fonts by copying (as administrator) src/fonts/Droid_Sans/DroidSans.ttf c:/Windows/Fonts
- Copy the out.txt as a out-win-powershell.txt into ref/ folder 
  PowerShell dont handle single apostrophs (ie: Can't) and marks the out.txt as a diff which is reported as error
- Make sure some commands in some test don't return || true
  testsuite/oiiotool/run.py
  I think for now just copy the output as this does not require changing runner python code and potentially make it fail in Bash



################################

# PNG - building problem in Docker

mkdir -p /opt/oiio/__build_png
cd /opt/oiio/__build_png
git remote add zach-png https://github.com/zachlewis/OpenImageIO.git
git pull zach-png
git checkout zach-png/build_PNG
git checkout 144f201ed97026998a53fce4b2da37a8445cbf73


# Bulding with updatedd PNG - needs to add Generato (Ninja) to the dependency builds 

- While building Dependnencies (and updating (Awk) files ) dont use VSCode as it messes up CRLF
- It is is better to connect from Emacs using Tramp mode:
/docker:hopeful_wozniak:/opt/oiio/src/cmake/ 

```
else ()
set (_pkg_cmake_verbose -DCMAKE_GENERATOR=Ninja -DCMAKE_VERBOSE_MAKEFILE=OFF
                        -DCMAKE_MESSAGE_LOG_LEVEL=ERROR
```


```

### First build
cmake .. -DOIIO_DOWNLOAD_MISSING_TESTDATA=ON -DOpenImageIO_BUILD_MISSING_DEPS=all -DINSTALL_FONTS=ON  -DCMAKE_BUILD_TYPE=Release

#### Python venv (numpy)
source ../../venvs/oiio/bin/activate

### Run Tests
CMAKE_PREFIX_PATH=$(pwd)/../dist/ PYTHONPATH=$(pwd)/lib/python/site-packages ctest -E broken --force-new-ctest-process --output-on-failure

```

# Building PNG locally
cd deps/PNG-build

cmake .. -DPNG_SHARED=OFF -DPNG_STATIC=ON -DPNG_EXECUTABLES=OFF -DPNG_TESTS=OFF -DPNG_TOOLS=OFF -DPNG_FRAMEWORK=OFF -DCMAKE_POSITION_INDEPENDENT_CODE=ON -DCMAKE_INSTALL_LIBDIR=lib -DPNG_PREFIX=oiio

## Usefull cmake stdout 
set(CMAKE_EXECUTE_PROCESS_COMMAND_ECHO STDOUT)


