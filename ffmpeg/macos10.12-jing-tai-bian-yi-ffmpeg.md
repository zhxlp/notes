# MacOS10.12静态编译FFmpeg

```bash

BUILD_DIR=/opt/ffbuild/source
TARGET_DIR=/opt/ffbuild/target

mkdir -p $BUILD_DIR
mkdir -p $TARGET_DIR/bin
mkdir -p $TARGET_DIR/lib
mkdir -p $TARGET_DIR/lib64

export LDFLAGS="-L${TARGET_DIR}/lib -L${TARGET_DIR}/lib64"
export CFLAGS="-I${TARGET_DIR}/include $LDFLAGS"
export CPPFLAGS="-I${TARGET_DIR}/include $LDFLAGS"
export PKG_CONFIG_PATH="$TARGET_DIR/lib/pkgconfig:$TARGET_DIR/lib64/pkgconfig"
export PATH="${TARGET_DIR}/bin:${PATH}"



# yasm-1.3.0.tar.gz
# http://www.tortall.net/projects/yasm/releases/yasm-1.3.0.tar.gz
cd $BUILD_DIR
tar -xvf yasm-1.3.0.tar.gz
cd yasm-1.3.0
./configure --prefix=$TARGET_DIR
make -j$(nproc)
make install


# nasm-2.14.02.tar.gz
# https://www.nasm.us/pub/nasm/releasebuilds/2.14.02/nasm-2.14.02.tar.gz
cd $BUILD_DIR
tar -xvf nasm-2.14.02.tar.gz
cd nasm-2.14.02
./configure --prefix=$TARGET_DIR
make -j$(nproc)
make install




# zlib-1.2.11.tar.gz
# http://zlib.net/zlib-1.2.11.tar.gz
cd $BUILD_DIR
tar -xvf zlib-1.2.11.tar.gz
cd zlib-1.2.11
./configure --prefix=$TARGET_DIR --static
make -j$(nproc)
make install


# xz-5.2.4.tar.xz
# https://sourceforge.net/projects/lzmautils/files/xz-5.2.4.tar.xz/download
cd $BUILD_DIR
tar -xvf xz-5.2.4.tar.xz
cd xz-5.2.4
./configure --prefix=$TARGET_DIR --disable-shared --enable-static
make -j$(nproc)
make install


# bzip2-1.0.6.tar.gz
https://ftp.osuosl.org/pub/clfs/conglomeration/bzip2/bzip2-1.0.6.tar.gz
cd $BUILD_DIR
tar -xvf bzip2-1.0.6.tar.gz
cd bzip2-1.0.6
make -j$(nproc)
make install PREFIX=$TARGET_DIR


# libiconv-1.15.tar.gz
# https://ftp.gnu.org/pub/gnu/libiconv/libiconv-1.15.tar.gz
cd $BUILD_DIR
tar -xvf libiconv-1.15.tar.gz
cd libiconv-1.15
./configure --prefix=$TARGET_DIR --enable-extra-encodings --disable-shared --enable-static
make -j$(nproc)
make install



# libtool-2.4.6.tar.gz
https://ftp.gnu.org/gnu/libtool/libtool-2.4.6.tar.gz
cd $BUILD_DIR
tar -xvf libtool-2.4.6.tar.gz
cd libtool-2.4.6
./configure --prefix=$TARGET_DIR --disable-shared --enable-static 
make -j$(nproc)
make install




# libxml2-2.9.9.tar.gz
# https://github.com/GNOME/libxml2/archive/refs/tags/v2.9.9.tar.gz
cd $BUILD_DIR
tar -xvf libxml2-2.9.9.tar.gz
cd libxml2-2.9.9
./autogen.sh
./configure --prefix=$TARGET_DIR --without-python --disable-shared --enable-static
make -j$(nproc)
make install



# openssl-1.0.2q.tar.gz
# https://www.openssl.org/source/openssl-1.0.2q.tar.gz
cd $BUILD_DIR
tar -xvf openssl-1.0.2q.tar.gz
cd openssl-1.0.2q
./Configure --prefix=$TARGET_DIR no-ssl2 no-ssl3 no-zlib no-shared enable-cms darwin64-x86_64-cc 
make -j$(nproc)
make install_sw




# libpng-1.6.36.tar.xz
# https://downloads.sourceforge.net/libpng/libpng-1.6.36.tar.xz
cd $BUILD_DIR
tar -xvf libpng-1.6.36.tar.xz
cd libpng-1.6.36
./configure --prefix=$TARGET_DIR --disable-shared --enable-static
make -j$(nproc)
make install




# freetype-2.9.1.tar.gz
# https://downloads.sourceforge.net/project/freetype/freetype2/2.9.1/freetype-2.9.1.tar.gz
cd $BUILD_DIR
tar -xvf freetype-2.9.1.tar.gz
cd freetype-2.9.1
./configure --prefix=$TARGET_DIR --disable-shared --enable-static --enable-freetype-config --without-harfbuzz
make -j$(nproc)
make install



# fribidi-1.0.5.tar.bz2
# https://github.com/fribidi/fribidi/releases/download/v1.0.5/fribidi-1.0.5.tar.bz2
cd $BUILD_DIR
tar -xvf fribidi-1.0.5.tar.bz2
cd fribidi-1.0.5
./configure --prefix=$TARGET_DIR --disable-debug --disable-shared --enable-static 
make -j$(nproc)
make install


# gmp-6.1.2.tar.xz
# https://gmplib.org/download/gmp/gmp-6.1.2.tar.xz
cd $BUILD_DIR
tar -xvf gmp-6.1.2.tar.xz
cd gmp-6.1.2
./configure --prefix=$TARGET_DIR --disable-shared --enable-static --with-pic
make -j$(nproc)
make install



# libogg-1.3.3.tar.gz
# https://github.com/xiph/ogg/releases/download/v1.3.3/libogg-1.3.3.tar.gz
cd $BUILD_DIR
tar -xvf libogg-1.3.3.tar.gz
cd libogg-1.3.3
./configure --prefix=$TARGET_DIR --enable-static --disable-shared
make -j$(nproc)
make install


# gperf-3.1.tar.gz
# https://ftp.gnu.org/gnu/gperf/gperf-3.1.tar.gz
cd $BUILD_DIR
tar -xvf gperf-3.1.tar.gz
cd gperf-3.1
./configure --prefix=$TARGET_DIR --enable-static --disable-shared 
make -j$(nproc)
make install


# fontconfig-2.13.91.tar.gz
# https://www.freedesktop.org/software/fontconfig/release/fontconfig-2.13.91.tar.gz
cd $BUILD_DIR
tar -xvf fontconfig-2.13.91.tar.gz
cd fontconfig-2.13.91
./configure --prefix=$TARGET_DIR --disable-docs --enable-libxml2 --enable-iconv --disable-shared --enable-static 
make -j$(nproc) 
make install 


# libffi-3.2.1.tar.gz
# https://sourceware.org/pub/libffi/libffi-3.2.1.tar.gz
cd $BUILD_DIR
tar -xvf libffi-3.2.1.tar.gz
cd libffi-3.2.1
./configure --prefix=$TARGET_DIR --disable-debug --disable-shared --enable-static 
make -j$(nproc) 
make install 



#   # gettext-0.19.8.1.tar.xz
#   # https://ftp.gnu.org/gnu/gettext/gettext-0.19.8.1.tar.xz
#   cd $BUILD_DIR
#   tar -xvf gettext-0.19.8.1.tar.xz
#   cd gettext-0.19.8.1
#   ./configure --prefix=$TARGET_DIR --disable-debug --disable-shared --enable-static --with-included-gettext --with-included-glib --with-included-libcroco --with-included-libunistring --with-emacs  --disable-java --disable-csharp --without-git --without-cvs --without-xz
#   make -j$(nproc) 
#   make install 
#   
#   
#   
#   
#   # glib-2.58.3.tar.xz
#   # https://download.gnome.org/sources/glib/2.58/glib-2.58.3.tar.xz
#   cd $BUILD_DIR
#   tar -xvf  glib-2.58.3.tar.xz
#   cd glib-2.58.3
#   ./autogen.sh
#   ./configure --prefix=$TARGET_DIR --disable-debug --disable-shared --enable-static
#   make -j$(nproc) 
#   make install 




# harfbuzz-1.7.5.tar.bz2
# https://www.freedesktop.org/software/harfbuzz/release/harfbuzz-1.7.5.tar.bz2
cd $BUILD_DIR
tar -xvf harfbuzz-1.7.5.tar.bz2
cd harfbuzz-1.7.5
./configure --prefix=$TARGET_DIR --disable-shared --enable-static  --without-glib 
make -j$(nproc)
make install



# libvorbis-1.3.6.tar.gz
# https://ftp.osuosl.org/pub/xiph/releases/vorbis/libvorbis-1.3.6.tar.gz
cd $BUILD_DIR
tar -xvf libvorbis-1.3.6.tar.gz
cd libvorbis-1.3.6
./configure --prefix=$TARGET_DIR --enable-static --disable-shared --disable-oggtest
make -j$(nproc)
make install



# aom-cb1d48da8da2061e72018761788a18b8fa8013bb.tar.gz
# https://aomedia.googlesource.com/aom/+archive/cb1d48da8da2061e72018761788a18b8fa8013bb.tar.gz
cd $BUILD_DIR
mkdir aom-2.0.2
cp -f aom-cb1d48da8da2061e72018761788a18b8fa8013bb.tar.gz aom-2.0.2
cd aom-2.0.2
tar -xvf aom-cb1d48da8da2061e72018761788a18b8fa8013bb.tar.gz
mkdir cmbuild
cd cmbuild
cmake .. -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX="$TARGET_DIR" -DBUILD_SHARED_LIBS=OFF -DENABLE_EXAMPLES=NO -DENABLE_TESTS=NO -DENABLE_TOOLS=NO -DENABLE_DOCS=OFF
make -j$(nproc)
make install



# libass-0.14.0.tar.gz
# https://github.com/libass/libass/releases/download/0.14.0/libass-0.14.0.tar.gz
cd $BUILD_DIR
tar -xvf libass-0.14.0.tar.gz
cd libass-0.14.0
./configure --prefix=$TARGET_DIR --disable-shared --enable-static --disable-coretext
make -j$(nproc)
make install



# lame-3.100.tar.gz
# https://sourceforge.net/projects/lame/files/lame/3.100/lame-3.100.tar.gz/download
cd $BUILD_DIR
tar -xvf lame-3.100.tar.gz
cd lame-3.100
./configure --prefix=$TARGET_DIR --disable-debug --disable-shared --enable-static --enable-nasm --disable-gtktest --disable-cpml --disable-frontend
make -j$(nproc)
make install

# opus-1.3.tar.gz
# https://archive.mozilla.org/pub/opus/opus-1.3.tar.gz
cd $BUILD_DIR
tar -xvf opus-1.3.tar.gz
cd opus-1.3
./configure --prefix=$TARGET_DIR --disable-shared --enable-static --disable-extra-programs --disable-doc
make -j$(nproc)
make install


# libtheora-1.1.1.tar.gz
# https://ftp.osuosl.org/pub/xiph/releases/theora/libtheora-1.1.1.tar.gz
cd $BUILD_DIR
tar -xvf libtheora-1.1.1.tar.gz
cd libtheora-1.1.1
./configure --prefix=$TARGET_DIR --disable-shared --enable-static --disable-examples --disable-oggtest --disable-vorbistest --disable-spec --disable-doc
make -j$(nproc)
make install


# libvpx-1.8.0.tar.gz
# https://download.videolan.org/pub/contrib/vpx/libvpx-1.8.0.tar.gz
cd $BUILD_DIR
tar -xvf libvpx-1.8.0.tar.gz
cd libvpx-1.8.0
./configure --prefix=$TARGET_DIR --disable-shared --enable-static --disable-examples --disable-tools --disable-docs --disable-unit-tests 
make -j$(nproc)
make install


# libwebp-1.0.2.tar.gz
# http://storage.googleapis.com/downloads.webmproject.org/releases/webp/libwebp-1.0.2.tar.gz
cd $BUILD_DIR
tar -xvf libwebp-1.0.2.tar.gz
cd libwebp-1.0.2
./configure --prefix=$TARGET_DIR --disable-shared --enable-static --enable-libwebpmux --disable-libwebpextras --disable-libwebpdemux --disable-sdl --disable-gl --disable-png --disable-jpeg --disable-tiff --disable-gif
make -j$(nproc)
make install


# opencore-amr-0.1.5.tar.gz
# https://sourceforge.net/projects/opencore-amr/files/opencore-amr/opencore-amr-0.1.5.tar.gz/download
cd $BUILD_DIR
tar -xvf opencore-amr-0.1.5.tar.gz
cd opencore-amr-0.1.5
./configure --prefix=$TARGET_DIR --disable-shared --enable-static --enable-amrnb-encoder --enable-amrnb-decoder --disable-examples
make -j$(nproc)
make install


# openjpeg-2.3.0.tar.gz
# https://github.com/uclouvain/openjpeg/archive/refs/tags/v2.3.0.tar.gz
cd $BUILD_DIR
tar -xvf openjpeg-2.3.0.tar.gz
cd openjpeg-2.3.0
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX="$TARGET_DIR" -DBUILD_SHARED_LIBS=OFF -DBUILD_PKGCONFIG_FILES=ON -DBUILD_CODEC=OFF -DWITH_ASTYLE=OFF -DBUILD_TESTING=OFF
make -j$(nproc)
make install


# soxr-0.1.3-Source.tar.xz
# https://sourceforge.net/projects/soxr/files/soxr-0.1.3-Source.tar.xz/download
cd $BUILD_DIR
tar -xvf soxr-0.1.3-Source.tar.xz
cd soxr-0.1.3-Source
mkdir build
cd build
cmake .. -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX="$TARGET_DIR" -DWITH_OPENMP=OFF -DBUILD_TESTS=OFF -DBUILD_EXAMPLES=OFF -DBUILD_SHARED_LIBS=OFF
make -j$(nproc)
make install


# srt-1.3.4.tar.gz
# https://github.com/Haivision/srt/archive/refs/tags/v1.3.4.tar.gz
cd $BUILD_DIR
tar -xvf srt-1.3.4.tar.gz
cd srt-1.3.4
mkdir build
cd build
cmake .. -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX="$TARGET_DIR"  -DENABLE_SHARED=OFF -DENABLE_STATIC=ON -DENABLE_ENCRYPTION=ON -DENABLE_APPS=OFF
make -j$(nproc)
make install


# vid.stab-1.1.0.tar.gz
# https://github.com/georgmartius/vid.stab/archive/refs/tags/v1.1.0.tar.gz
cd $BUILD_DIR
tar -xvf vid.stab-1.1.0.tar.gz
cd vid.stab-1.1.0
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX="$TARGET_DIR" -DBUILD_SHARED_LIBS=OFF -DUSE_OMP=OFF
make -j$(nproc)
make install


# x264-0a84d986e7020f8344f00752e3600b9769cc1e85.tar.gz
# https://code.videolan.org/videolan/x264/-/archive/0a84d986e7020f8344f00752e3600b9769cc1e85/x264-0a84d986e7020f8344f00752e3600b9769cc1e85.tar.gz
cd $BUILD_DIR
tar -xvf x264-0a84d986e7020f8344f00752e3600b9769cc1e85.tar.gz
cd x264-0a84d986e7020f8344f00752e3600b9769cc1e85
./configure --prefix=$TARGET_DIR --enable-static --disable-opencl --disable-lavf --disable-swscale --disable-avs --disable-lsmash
make -j$(nproc)
make install


# x265_3.0.tar.gz
# https://bitbucket.org/multicoreware/x265_git/downloads/x265_3.0.tar.gz
cd $BUILD_DIR
tar -xvf x265_3.0.tar.gz
cd x265_3.0
mkdir cbuild
cd cbuild
cmake ../source -DCMAKE_INSTALL_PREFIX="$TARGET_DIR" -DCMAKE_BUILD_TYPE=Release  -DENABLE_SHARED=OFF -DENABLE_CLI=OFF 
make -j$(nproc)
make install


# zimg-release-2.8.tar.gz
# https://github.com/sekrit-twc/zimg/archive/release-2.8.tar.gz
cd $BUILD_DIR
tar -xvf zimg-release-2.8.tar.gz
cd zimg-release-2.8
./autogen.sh
./configure --prefix=$TARGET_DIR --enable-static --disable-shared 
make -j$(nproc)
make install


# speex-Speex-1.2.0.tar.gz
# https://github.com/xiph/speex/archive/refs/tags/Speex-1.2.0.tar.gz
cd $BUILD_DIR
tar -xvf speex-Speex-1.2.0.tar.gz
cd speex-Speex-1.2.0
./autogen.sh
./configure --prefix=$TARGET_DIR --enable-static --disable-shared 
make -j$(nproc)
make install



# vo-amrwbenc-0.1.3.tar.gz
# https://sourceforge.net/projects/opencore-amr/files/vo-amrwbenc/vo-amrwbenc-0.1.3.tar.gz/download
cd $BUILD_DIR
tar -xvf vo-amrwbenc-0.1.3.tar.gz
cd vo-amrwbenc-0.1.3
./configure --prefix=$TARGET_DIR --enable-static --disable-shared
make -j$(nproc)
make install



# xvidcore-1.3.5.tar.gz
# https://downloads.xvid.com/downloads/xvidcore-1.3.5.tar.gz
cd $BUILD_DIR
tar -xvf xvidcore-1.3.5.tar.gz
cd xvidcore/build/generic
./configure --prefix=$TARGET_DIR 
make -j$(nproc)
make install
rm -f "$TARGET_DIR"/lib/libxvidcore.4.dylib






# ffmpeg-4.4.tar.xz
# https://ffmpeg.org/releases/ffmpeg-4.4.tar.xz
cd $BUILD_DIR
rm -rf ffmpeg-4.4 && tar -xvf ffmpeg-4.4.tar.xz && cd ffmpeg-4.4
./configure --cc=/usr/bin/clang --prefix=$TARGET_DIR --enable-gpl --enable-version3 --enable-libfontconfig --enable-frei0r --enable-gmp --enable-libaom --enable-libfribidi --enable-libfreetype --enable-libmp3lame --enable-libopencore-amrnb --enable-libopencore-amrwb --enable-libopenjpeg --enable-libsoxr --enable-libspeex --enable-libsrt --enable-libvorbis --enable-libopus --enable-libtheora --enable-libvidstab --enable-libvo-amrwbenc --enable-libvpx --enable-libwebp --enable-libx264 --enable-libx265 --enable-libxml2 --enable-libxvid --enable-libzimg --enable-libass --enable-openssl --enable-nonfree --disable-autodetect --disable-doc --pkg-config-flags="--static" --extra-cflags="-I$TARGET_DIR/include" --extra-ldflags="-L$TARGET_DIR/lib" --extra-ldexeflags="-Bstatic"
make -j$(nproc)
make install




./configure --cc=/usr/bin/clang --prefix=$TARGET_DIR --enable-gpl --enable-version3 --enable-libfontconfig --enable-frei0r --enable-gmp --enable-libaom --enable-libfribidi --enable-libfreetype --enable-libmp3lame --enable-libopencore-amrnb --enable-libopencore-amrwb --enable-libopenjpeg --enable-libsoxr --enable-libspeex --enable-libsrt --enable-libvorbis --enable-libopus --enable-libtheora --enable-libvidstab --enable-libvo-amrwbenc --enable-libvpx --enable-libwebp --enable-libx264 --enable-libx265 --enable-libxml2 --enable-libxvid --enable-libzimg --enable-libass --disable-autodetect --disable-doc --pkg-config-flags="--static" --extra-cflags="-I$TARGET_DIR/include" --extra-ldflags="-L$TARGET_DIR/lib" --extra-ldexeflags="-Bstatic"



```
