Welcome! This repo serves as the landing page and discussion area for getting involved in this project.

## Setup
Manual:
https://github.com/neuroAtlas/WlzIIPSrv/tree/master/doc/manual

To build the server you need to get the following repos

https://github.com/neuroAtlas/External/tree/master
https://github.com/neuroAtlas/Woolz/tree/release-1.5.0
https://github.com/neuroAtlas/WlzIIPSrv/tree/release-1.1.1

I usually build and install all the software into a directory tree
out of the /usr tree with prefix=/opt/MouseAtlas, but any other
prefix should be fine. I don't advise installing into the /usr
directory tree as there are specific versions of jpeg and tiff
which may conflict.

There's a table of which External packages are needed for which
build in the External README. I assume you're building the IIP3D
server (WlzIIPSrv) for which you'll also need Woolz (our image
processing library, pronounced wool's and often abbreviated to wlz).
Woolz has a number of small command line programs that you would
need to convert/build Woolz objects from which the server cuts
tiles. You'll also need the Woolz libraries to build the server.

Each of the External files has a README in it which you can use
to build the software, or alternatively you could use something
like:
```
Jpeg:
  cd /opt/MouseAtlas/
  export MA=/opt/MouseAtlas
  export PATH="$MA"/bin:$PATH
  export LD_LIBRARY_PATH="$MA"/lib64:$LD_LIBRARY_PATH
  mkdir -p src/External/Jpeg
  cd src/External/Jpeg/
  tar -zxf /opt/websrc/downloads/jpegsrc.v8d.tar.gz
  cd jpeg-8d/
  ./configure --prefix $MA --disable-shared --enable-static --with-pic
  make
  make install
Tiff:
  cd /opt/MouseAtlas/
  export MA=/opt/MouseAtlas
  export PATH="$MA"/bin:$PATH
  export LD_LIBRARY_PATH="$MA"/lib64:$LD_LIBRARY_PATH
  mkdir -p src/External/Tiff
  cd src/External/Tiff/
  tar -zxf /opt/websrc/downloads/tiff-3.9.5.tar.gz
  cd tiff-3.9.5/
  ./configure --prefix $MA --disable-shared --enable-static --with-pic \
              --with-jpeg-include-dir=$MA/include --with-jpeg-lib-dir==$MA/lib64
  make
  make install
libpng:
  cd /opt/MouseAtlas/
  export MA=/opt/MouseAtlas
  export PATH="$MA"/bin:$PATH
  export LD_LIBRARY_PATH="$MA"/lib64:$LD_LIBRARY_PATH
  mkdir -p src/External/PNG
  cd src/External/PNG/
  tar -zxf /opt/websrc/downloads/libpng-1.2.50.tar.gz
  cd libpng-1.2.50/
  ./configure --prefix $MA --disable-shared --enable-static --with-pic
  make
  make install
Log4Cpp:
  cd /opt/MouseAtlas/
  export MA=/opt/MouseAtlas
  export PATH="$MA"/bin:$PATH
  export LD_LIBRARY_PATH="$MA"/lib64:$LD_LIBRARY_PATH
  mkdir -p src/External/Log4Cpp
  cd src/External/Log4Cpp/
  tar -zxf /opt/websrc/downloads/log4cpp-1.0.tar.gz
  patch -p0 < /opt/websrc/downloads/log4cpp-1.0.patch
  cd log4cpp-1.0/
  ./configure --prefix $MA --disable-shared --enable-static --with-pic
  make
  make install
NIfTI:
  cd /opt/MouseAtlas/
  export MA=/opt/MouseAtlas
  export PATH="$MA"/bin:$PATH
  export LD_LIBRARY_PATH="$MA"/lib64:$LD_LIBRARY_PATH
  mkdir -p src/External/NIfTI
  cd src/External/NIfTI/
  tar -zxf /opt/websrc/downloads/nifticlib-2.0.0.tar.gz
  cd nifticlib-2.0.0/
  CFLAGS='-fPIC' cmake -DCMAKE_INSTALL_PREFIX=/opt/MouseAtlas \
         -DBUILD_TESTING=OFF -DBUILD_SHARED_LIBS=OFF -DBUILD_SHARED_LIBS=OFF
  make
  make install
  cd /opt/MouseAtlas/include/nifti/; ln *.h ..
```
The External repo still doesn't include the fcgi lib, so you'll
need to download this and build it, eg:
```
Fcgi:
  mkdir /some/where/fcgi-2.4.0
  cd /opt/; ln -s /some/where/fcgi-2.4.0 fcgi
  cd /opt/fcgi/
  mkdir src
  cd src/
  tar -zxf /some/where/downloads/fcgi-2.4.0.tar.gz
  patch -p0 < /some/where/downloads/fcgi-2.4.0-patch
  cd fcgi-2.4.0/
  ./configure --prefix=/opt/fcgi
  make
  make install
  ln -s /opt/fcgi/lib64 /opt/fcgi/lib
```

When building Woolz (again in the Woolz directory there's a build.sh
file with common ./configure setups). You're probably not going to
need JavaWlz which will save some hassle. JavaWlz generates a Java
binding to the Woolz libraries via the JNI (there's a Bioformats
Woolz reader/writer which uses this) allowing a subset of Woolz
objects to be read directly into anything using Bioformats (eg
ImageJ, OMERO, etc...). As an alternative to the build.sh file
you could use something like:
```
Woolz:
  cd /opt/MouseAtlas/src/
  export MA=/opt/MouseAtlas
  export PATH="$MA"/bin:$PATH
  export LD_LIBRARY_PATH="$MA"/lib64:$LD_LIBRARY_PATH
  git clone https://github.com/ma-tech/Woolz.git
  cd Woolz/
  git checkout master
  autoreconf -fi
  ./configure --prefix=$MA  --enable-optimise --enable-openmp --enable-extff \
              --disable-shared --enable-static --with-pic \
              --with-jpeg=$MA --with-tiff=$MA --with-nifti=$MA
  make -j8
  make install
```
To build the Woolz (IIP3D) server you could again use the build
script or you could use something like:
```
WlzIIPSrv:
  cd /opt/MouseAtlas/src/
  export MA=/opt/MouseAtlas
  export PATH="$MA"/bin:$PATH
  export LD_LIBRARY_PATH="$MA"/lib64:$LD_LIBRARY_PATH
  git clone https://github.com/ma-tech/WlzIIPSrv.git
  cd WlzIIPSrv/
  git checkout master
  autoreconf -fi
  export CFLAGS='-O3 -msse3'
  export CXXFLAGS='-O3 -msse3'
  ./configure --prefix=$MA \
              --with-fcgi-incl=/opt/fcgi/include --with-fcgi-lib=/opt/fcgi/lib \
              --with-jpeg-includes=$MA/include --with-jpeg-libraries=$MA/lib \
              --with-tiff-includes=$MA/include --with-tiff-libraries=$MA/lib \
              --with-png-includes=$MA/include --with-png-libraries=$MA/lib \
              --with-nifti-incl=$MA/include --with-nifti-lib=$MA/lib \
              --with-log4cpp-incl=$MA/include --with-log4cpp-lib=$MA/lib \
              --with-wlz-incl=$MA/include --with-wlz-lib=$MA/lib \
              --enable-openmp

```
Note that the fcgi executable is left (in this case) in the
source tree as:
 /opt/MouseAtlas/src/WlzIIPSrv/src/wlziipsrv.fcgi

Also within the server repo are system init/configuration
scripts, eg

https://github.com/ma-tech/WlzIIPSrv/blob/master/wlziipsrv.service
and
https://github.com/ma-tech/WlzIIPSrv/blob/master/wlziipsrv.env

for a systemd based OS.

More details on the server and it's environment variables can be
found in the manual.

Once you have the external software built then Woolz and WlzIIPSrv
_should_ build fairly easily, although sometimes people have
not noticed that flex and bison can be needed to build WlzIIPSrv.

Once you have the server installed it should be fairly easy to test
using simple urls, eg:

http://localhost/fcgi-bin/wlziipsrv.fcgi?WLZ=/opt/MAWWW/public/Wlz/test.wlz&CVT=jpeg

where /opt/MAWWW/public/Wlz/test.wlz is the full path to your
Woolz file.

