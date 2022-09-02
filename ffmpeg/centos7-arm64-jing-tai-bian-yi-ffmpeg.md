# CentOS7 Arm64静态编译FFmpeg

```bash

curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo

yum install -y https://mirrors.aliyun.com/epel/epel-release-latest-7.noarch.rpm
sed -i 's|^#baseurl=http://download.fedoraproject.org/pub|baseurl=https://mirrors.aliyun.com|' /etc/yum.repos.d/epel*
sed -i 's|^metalink|#metalink|' /etc/yum.repos.d/epel*

yum install -y autoconf automake cmake gcc  make pkgconfig gcc-c++ libtool cmake3 meson bzip2 glibc-static libstdc++-static

#!/bin/bash
set -e

BUILD_DIR=/opt/ffbuild/source
TARGET_DIR=/opt/ffbuild/target

mkdir -p $BUILD_DIR
mkdir -p $TARGET_DIR/bin
mkdir -p $TARGET_DIR/lib
mkdir -p $TARGET_DIR/lib64
mkdir -p $TARGET_DIR/syslib

cp -f /usr/lib/gcc/aarch64-redhat-linux/4.8.2/libgcc.a $TARGET_DIR/syslib/
cp -f /usr/lib/gcc/aarch64-redhat-linux/4.8.2/libgomp.a $TARGET_DIR/syslib/
cp -f /usr/lib/gcc/aarch64-redhat-linux/4.8.2/libstdc++.a $TARGET_DIR/syslib/
cp -f /usr/lib64/librt.a $TARGET_DIR/syslib/


# -static-libgcc 
export LDFLAGS="-static-libgcc -static-libstdc++ -L${TARGET_DIR}/lib -L${TARGET_DIR}/lib64 -L${TARGET_DIR}/syslib"
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


# nasm-2.15.05.tar.gz
# https://www.nasm.us/pub/nasm/releasebuilds/2.15.05/nasm-2.15.05.tar.gz
cd $BUILD_DIR
tar -xvf nasm-2.15.05.tar.gz
cd nasm-2.15.05
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


# xz-5.2.5.tar.xz
# https://sourceforge.net/projects/lzmautils/files/xz-5.2.5.tar.xz/download
cd $BUILD_DIR
tar -xvf xz-5.2.5.tar.xz
cd xz-5.2.5
./configure --prefix=$TARGET_DIR --disable-shared --enable-static
make -j$(nproc)
make install


# libiconv-1.16.tar.gz
# https://ftp.gnu.org/pub/gnu/libiconv/libiconv-1.16.tar.gz
cd $BUILD_DIR
tar -xvf libiconv-1.16.tar.gz
cd libiconv-1.16
./configure --prefix=$TARGET_DIR --enable-extra-encodings --disable-shared --enable-static
make -j$(nproc)
make install


# libxml2-2.9.12.tar.gz
# https://github.com/GNOME/libxml2/archive/refs/tags/v2.9.12.tar.gz
cd $BUILD_DIR
tar -xvf libxml2-2.9.12.tar.gz
cd libxml2-2.9.12
./autogen.sh
./configure --prefix=$TARGET_DIR --without-python --disable-shared --enable-static
make -j$(nproc)
make install


# openssl-1.0.2k.tar.gz
# https://www.openssl.org/source/old/1.0.2/openssl-1.0.2k.tar.gz
cd $BUILD_DIR
tar -xvf openssl-1.0.2k.tar.gz
cd openssl-1.0.2k
CC="gcc $CFLAGS $LDFLAGS" ./Configure threads zlib no-shared enable-camellia enable-ec enable-srp linux-aarch64 --prefix=$TARGET_DIR
make -j$(nproc)
make install_sw


# libpng-1.6.37.tar.xz
# https://downloads.sourceforge.net/libpng/libpng-1.6.37.tar.xz
cd $BUILD_DIR
tar -xvf libpng-1.6.37.tar.xz
cd libpng-1.6.37
./configure --prefix=$TARGET_DIR --disable-shared --enable-static
make -j$(nproc)
make install


# gavl-1.4.0.tar.gz
# https://downloads.sourceforge.net/gmerlin/gavl-1.4.0.tar.gz
cd $BUILD_DIR
tar -xvf gavl-1.4.0.tar.gz
cd gavl-1.4.0
./configure --prefix=$TARGET_DIR --disable-shared --enable-static --without-doxygen --build=arm-linux
make -j$(nproc)
make install


# freetype-2.8.1.tar.gz
# https://sourceforge.net/projects/freetype/files/freetype2/2.8.1/freetype-2.8.1.tar.gz/download
cd $BUILD_DIR
tar -xvf freetype-2.8.1.tar.gz
cd freetype-2.8.1
./configure --prefix=$TARGET_DIR --disable-shared --enable-static --without-harfbuzz
make -j$(nproc)
make install


# fribidi-1.0.10.tar.xz
# https://github.com/fribidi/fribidi/releases/download/v1.0.10/fribidi-1.0.10.tar.xz
cd $BUILD_DIR
tar -xvf fribidi-1.0.10.tar.xz
cd fribidi-1.0.10
./configure --prefix=$TARGET_DIR --disable-shared --enable-static 
make -j$(nproc)
make install


# gmp-6.2.1.tar.xz
# https://ftp.gnu.org/gnu/gmp/gmp-6.2.1.tar.xz
cd $BUILD_DIR
tar -xvf gmp-6.2.1.tar.xz
cd gmp-6.2.1
./configure --prefix=$TARGET_DIR --disable-shared --enable-static 
make -j$(nproc)
make install



# frei0r-plugins-1.7.0.tar.gz
# https://files.dyne.org/frei0r/releases/frei0r-plugins-1.7.0.tar.gz
cd $BUILD_DIR
tar -xvf frei0r-plugins-1.7.0.tar.gz
cd frei0r-plugins-1.7.0
./configure --prefix=$TARGET_DIR --disable-shared --enable-static
/usr/bin/cp -f README.txt README.md
/usr/bin/cp -f ChangeLog.txt ChangeLog
/usr/bin/cp -f TODO.txt TODO
/usr/bin/cp -f AUTHORS.txt AUTHORS
make -j$(nproc)
sed -i 's#\.so#\.a#g' src/Makefile
make install



# libogg-1.3.4.tar.gz
# https://github.com/xiph/ogg/releases/download/v1.3.4/libogg-1.3.4.tar.gz
cd $BUILD_DIR
tar -xvf libogg-1.3.4.tar.gz
cd libogg-1.3.4
./configure --prefix=$TARGET_DIR --enable-static --disable-shared
make -j$(nproc)
make install



# libuuid-1.0.3.tar.gz
# https://sourceforge.net/projects/libuuid/files/libuuid-1.0.3.tar.gz/download
cd $BUILD_DIR
tar -xvf libuuid-1.0.3.tar.gz
cd libuuid-1.0.3
./configure --prefix=$TARGET_DIR --enable-static --disable-shared
make -j$(nproc)
make install


# gperf-3.0.4.tar.gz
# https://ftp.gnu.org/gnu/gperf/gperf-3.0.4.tar.gz
cd $BUILD_DIR
tar -xvf gperf-3.0.4.tar.gz
cd gperf-3.0.4
./configure --prefix=$TARGET_DIR --enable-static --disable-shared 
make -j$(nproc)
make install


# fontconfig-2.13.94.tar.gz
# https://www.freedesktop.org/software/fontconfig/release/fontconfig-2.13.94.tar.gz
cd $BUILD_DIR
tar -xvf fontconfig-2.13.94.tar.gz
cd fontconfig-2.13.94
./configure --prefix=$TARGET_DIR --disable-docs --enable-libxml2 --enable-iconv --disable-shared --enable-static
make -j$(nproc)
make install


# harfbuzz-1.7.5.tar.bz2
# https://www.freedesktop.org/software/harfbuzz/release/harfbuzz-1.7.5.tar.bz2
cd $BUILD_DIR
tar -xvf harfbuzz-1.7.5.tar.bz2
cd harfbuzz-1.7.5
./configure --prefix=$TARGET_DIR --disable-shared --enable-static 
make -j$(nproc)
make install


# libvorbis-1.3.3.tar.gz
# https://ftp.osuosl.org/pub/xiph/releases/vorbis/libvorbis-1.3.3.tar.gz
cd $BUILD_DIR
tar -xvf libvorbis-1.3.3.tar.gz
cd libvorbis-1.3.3
./configure --prefix=$TARGET_DIR --enable-static --disable-shared --disable-oggtest --build=arm-linux
make -j$(nproc)
make install


# arm 不支持
# vmaf-1.5.2.tar.gz
# https://github.com/Netflix/vmaf/archive/refs/tags/v1.5.2.tar.gz
# cd $BUILD_DIR
# tar -xvf vmaf-1.5.2.tar.gz
# cd $BUILD_DIR/vmaf-1.5.2
# mkdir build
# cd build
# meson --prefix="$TARGET_DIR" --buildtype=release --default-library=static -Dbuilt_in_models=true -Denable_tests=false -Denable_docs=false -Denable_avx512=true -Denable_float=true ../libvmaf
# ninja-build -j"$(nproc)"
# ninja-build install
# rm -rf $TARGET_DIR/lib64/libvmaf.so*
# # /usr/bin/cp -f $TARGET_DIR/include/libvmaf/libvmaf.h $TARGET_DIR/include/libvmaf.h 
# sed -i 's/Libs.private:/Libs.private: -lstdc++/' $TARGET_DIR/lib64/pkgconfig/libvmaf.pc


# 无法编译 未知
# aom-cb1d48da8da2061e72018761788a18b8fa8013bb.tar.gz
# https://aomedia.googlesource.com/aom/+archive/cb1d48da8da2061e72018761788a18b8fa8013bb.tar.gz
# cd $BUILD_DIR
# mkdir aom-2.0.2
# /usr/bin/cp -f aom-cb1d48da8da2061e72018761788a18b8fa8013bb.tar.gz aom-2.0.2
# cd aom-2.0.2
# tar -xvf aom-cb1d48da8da2061e72018761788a18b8fa8013bb.tar.gz
# mkdir cmbuild
# cd cmbuild
# cmake3 -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX="$TARGET_DIR" -DBUILD_SHARED_LIBS=OFF -DENABLE_EXAMPLES=NO -DENABLE_TESTS=NO -DENABLE_TOOLS=NO -DENABLE_DOCS=OFF -DCONFIG_AV1_ENCODER=0  ..
# make -j$(nproc)
# make install
# echo "Requires.private: libvmaf" >> $TARGET_DIR/lib64/pkgconfig/aom.pc


# dav1d-0.5.2.tar.gz
# https://code.videolan.org/videolan/dav1d/-/archive/0.5.2/dav1d-0.5.2.tar.gz
cd $BUILD_DIR
tar -xvf dav1d-0.5.2.tar.gz
cd dav1d-0.5.2
mkdir build
cd build
meson --prefix="$TARGET_DIR" --buildtype=release --default-library=static ..
ninja-build -j$(nproc)
ninja-build install




# game-music-emu-0.6.2.tar.xz
# https://bitbucket.org/mpyne/game-music-emu/downloads/game-music-emu-0.6.2.tar.xz
cd $BUILD_DIR
tar -xvf game-music-emu-0.6.2.tar.xz
cd game-music-emu-0.6.2
mkdir build
cd build
cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX="$TARGET_DIR" -DBUILD_SHARED_LIBS=OFF ..
make -j$(nproc)
make install



# libass-0.13.4.tar.gz
# https://github.com/libass/libass/releases/download/0.13.4/libass-0.13.4.tar.gz
cd $BUILD_DIR
tar -xvf libass-0.13.4.tar.gz
cd libass-0.13.4
./configure --prefix=$TARGET_DIR --disable-shared --enable-static 
make -j$(nproc)
make install


# lame-3.100.tar.gz
# https://sourceforge.net/projects/lame/files/lame/3.100/lame-3.100.tar.gz/download
cd $BUILD_DIR
tar -xvf lame-3.100.tar.gz
cd lame-3.100
./configure --prefix=$TARGET_DIR --disable-shared --enable-static --enable-nasm --disable-gtktest --disable-cpml --disable-frontend
make -j$(nproc)
make install


# opus-1.1.2.tar.gz
# https://github.com/xiph/opus/archive/refs/tags/v1.1.2.tar.gz
cd $BUILD_DIR
tar -xvf opus-1.1.2.tar.gz
cd opus-1.1.2
./autogen.sh 
./configure --prefix=$TARGET_DIR --disable-shared --enable-static --disable-extra-programs
make -j$(nproc)
make install


# libtheora-1.1.1.tar.gz
# https://ftp.osuosl.org/pub/xiph/releases/theora/libtheora-1.1.1.tar.gz
cd $BUILD_DIR
tar -xvf libtheora-1.1.1.tar.gz
cd libtheora-1.1.1
./configure --prefix=$TARGET_DIR --disable-shared --enable-static --disable-examples --disable-oggtest --disable-vorbistest --disable-spec --disable-doc --build=arm-linux
make -j$(nproc)
make install


# libvpx-1.4.0.tar.bz2
# https://download.videolan.org/pub/contrib/vpx/libvpx-1.4.0.tar.bz2
cd $BUILD_DIR
tar -xvf libvpx-1.4.0.tar.bz2
cd libvpx-1.4.0
./configure --prefix=$TARGET_DIR --disable-shared --enable-static --disable-examples --disable-docs --disable-unit-tests
make -j$(nproc)
make install


# libwebp-0.6.1.tar.gz
# http://storage.googleapis.com/downloads.webmproject.org/releases/webp/libwebp-0.6.1.tar.gz
cd $BUILD_DIR
tar -xvf libwebp-0.6.1.tar.gz
cd libwebp-0.6.1
./configure --prefix=$TARGET_DIR --disable-shared --enable-static --enable-libwebpmux --disable-libwebpextras --disable-libwebpdemux --disable-sdl --disable-gl --disable-png --disable-jpeg --disable-tiff --disable-gif
make -j$(nproc)
make install



# opencore-amr-0.1.3.tar.gz
# https://sourceforge.net/projects/opencore-amr/files/opencore-amr/opencore-amr-0.1.3.tar.gz/download
cd $BUILD_DIR
tar -xvf opencore-amr-0.1.3.tar.gz
cd opencore-amr-0.1.3
./configure --prefix=$TARGET_DIR --disable-shared --enable-static --enable-amrnb-encoder --enable-amrnb-decoder --disable-examples
make -j$(nproc)
make install


# openjpeg-2.4.0.tar.gz
# https://github.com/uclouvain/openjpeg/archive/refs/tags/v2.4.0.tar.gz
cd $BUILD_DIR
tar -xvf openjpeg-2.4.0.tar.gz
cd openjpeg-2.4.0
mkdir build && cd build
cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX="$TARGET_DIR" -DBUILD_SHARED_LIBS=OFF -DBUILD_PKGCONFIG_FILES=ON -DBUILD_CODEC=OFF -DWITH_ASTYLE=OFF -DBUILD_TESTING=OFF ..
make -j$(nproc)
make install


# soxr-0.1.2-Source.tar.xz
# https://sourceforge.net/projects/soxr/files/soxr-0.1.2-Source.tar.xz/download
cd $BUILD_DIR
tar -xvf soxr-0.1.2-Source.tar.xz
cd soxr-0.1.2-Source
mkdir build
cd build
cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX="$TARGET_DIR" -DWITH_OPENMP=OFF -DBUILD_TESTS=OFF -DBUILD_EXAMPLES=OFF -DBUILD_SHARED_LIBS=OFF ..
make -j$(nproc)
make install


# srt-1.4.3.tar.gz
# https://github.com/Haivision/srt/archive/refs/tags/v1.4.3.tar.gz
cd $BUILD_DIR
tar -xvf srt-1.4.3.tar.gz
cd srt-1.4.3
mkdir build
cd build
cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX="$TARGET_DIR"  -DENABLE_SHARED=OFF -DENABLE_STATIC=ON -DENABLE_ENCRYPTION=ON -DENABLE_APPS=OFF ..
make -j$(nproc)
make install


# vid.stab-1.1.0.tar.gz
# https://github.com/georgmartius/vid.stab/archive/refs/tags/v1.1.0.tar.gz
cd $BUILD_DIR
tar -xvf vid.stab-1.1.0.tar.gz
cd vid.stab-1.1.0
mkdir build && cd build
cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX="$TARGET_DIR" -DBUILD_SHARED_LIBS=OFF ..
make -j$(nproc)
make install


# x264-b86ae3c66f51ac9eab5ab7ad09a9d62e67961b8a.tar.gz
# https://code.videolan.org/videolan/x264/-/archive/b86ae3c66f51ac9eab5ab7ad09a9d62e67961b8a/x264-b86ae3c66f51ac9eab5ab7ad09a9d62e67961b8a.tar.gz
cd $BUILD_DIR
tar -xvf x264-b86ae3c66f51ac9eab5ab7ad09a9d62e67961b8a.tar.gz
cd x264-b86ae3c66f51ac9eab5ab7ad09a9d62e67961b8a
./configure --prefix=$TARGET_DIR --enable-static --disable-opencl --disable-lavf --disable-swscale --disable-avs
make -j$(nproc)
make install


# x265-3.4.tar.gz
# https://github.com/videolan/x265/archive/refs/tags/3.4.tar.gz
cd $BUILD_DIR
tar -xvf x265-3.4.tar.gz
cd x265-3.4
mkdir cbuild
cd cbuild
cmake -DCMAKE_INSTALL_PREFIX="$TARGET_DIR" -DCMAKE_BUILD_TYPE=Release  -DENABLE_SHARED=OFF -DENABLE_CLI=OFF -DCMAKE_ASM_NASM_FLAGS=-w-macro-params-legacy ../source
make -j$(nproc)
make install


# zimg-release-2.8.tar.gz
# https://github.com/sekrit-twc/zimg/archive/refs/tags/release-2.8.tar.gz
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


# xvidcore-1.3.7.tar.gz
# https://downloads.xvid.com/downloads/xvidcore-1.3.7.tar.gz
cd $BUILD_DIR
tar -xvf xvidcore-1.3.7.tar.gz
cd xvidcore/build/generic
./bootstrap.sh
./configure --prefix=$TARGET_DIR --enable-static --disable-shared
make -j$(nproc)
make install
rm -f "$TARGET_DIR"/lib/libxvidcore.so*

# zvbi-0.2.35.tar.bz2
# https://sourceforge.net/projects/zapping/files/zvbi/0.2.35/zvbi-0.2.35.tar.bz2/download
cd $BUILD_DIR
tar -xvf zvbi-0.2.35.tar.bz2
cd zvbi-0.2.35
./configure --prefix=$TARGET_DIR --enable-static --disable-shared --build=arm-linux
make -j$(nproc)
make install


# ffmpeg-4.4.tar.xz
# https://ffmpeg.org/releases/ffmpeg-4.4.tar.xz
cd $BUILD_DIR
rm -rf ffmpeg-4.4 && tar -xvf ffmpeg-4.4.tar.xz && cd ffmpeg-4.4
./configure --prefix=$TARGET_DIR --enable-gpl --enable-version3 --enable-static --disable-debug --disable-ffplay --disable-indev=sndio --disable-outdev=sndio --cc=gcc --enable-fontconfig --enable-frei0r --enable-gmp --enable-libgme --enable-gray --disable-libaom --enable-libfribidi --enable-libass --disable-libvmaf --enable-libfreetype --enable-libmp3lame --enable-libopencore-amrnb --enable-libopencore-amrwb --enable-libopenjpeg --enable-libsoxr --enable-libspeex --enable-libsrt --enable-libvorbis --enable-libopus --enable-libtheora --enable-libvidstab --enable-libvo-amrwbenc --enable-libvpx --enable-libwebp --enable-libx264 --enable-libx265 --enable-libxml2 --enable-libdav1d --enable-libxvid --enable-libzvbi --enable-libzimg --pkg-config-flags="--static" --extra-libs="-lpthread -lm -lz" --extra-ldexeflags="-static"
make -j$(nproc)
make install


# pkg-config --static --cflags --libs libavutil libswresample libswscale libavcodec libavformat libavdevice libavfilter


# -I/opt/ffbuild/target/include  -pthread /opt/ffbuild/target/lib/libiconv.a -L/opt/ffbuild/target/lib -L/opt/ffbuild/target/lib64 -lavdevice -lavfilter -lass -lharfbuzz -lfribidi -lvidstab -lgomp -lzimg -lfontconfig -lfreetype -lpng16 -lswscale -lpostproc -lavformat -lxml2 -lgme -lgmp -lsrt -lc -lssl -lcrypto -lavcodec -lvpx -lwebpmux -liconv -llzma -ldav1d -lopencore-amrwb -lzvbi -lpng -lz -laom -lvmaf -lmp3lame -lopencore-amrnb -lopenjp2 -lopus -lspeex -ltheoraenc -ltheoradec -lvo-amrwbenc -lvorbisenc -lvorbis -logg -lwebp -lx264 -lpthread -lx265 -lstdc++ -lrt -ldl -lxvidcore -lswresample -lsoxr -lavutil -lm 


















# nettle-3.7.3.tar.gz
# https://ftp.gnu.org/gnu/nettle/nettle-3.7.3.tar.gz
# cd $BUILD_DIR
# tar -xvf nettle-3.7.3.tar.gz
# cd nettle-3.7.3
# ./configure --prefix=$TARGET_DIR --disable-shared --enable-static --disable-openssl --disable-documentation
# make -j$(nproc)
# make install
# 
# 
# libtasn1-4.9.tar.gz
# https://ftp.gnu.org/gnu/libtasn1/libtasn1-4.9.tar.gz
# cd $BUILD_DIR
# tar -xvf libtasn1-4.9.tar.gz
# cd libtasn1-4.9
# ./configure --prefix=$TARGET_DIR --disable-shared --enable-static
# make -j$(nproc)
# make install
# 
# 
# libunistring-0.9.10.tar.gz
# https://ftp.gnu.org/gnu/libunistring/libunistring-0.9.10.tar.gz
# cd $BUILD_DIR
# tar -xvf libunistring-0.9.10.tar.gz
# cd libunistring-0.9.10
# ./configure --prefix=$TARGET_DIR --disable-shared --enable-static
# make -j$(nproc)
# make install
# 
# 
# 
# gnutls-3.6.16.tar.xz
# https://www.gnupg.org/ftp/gcrypt/gnutls/v3.6/gnutls-3.6.16.tar.xz
# cd $BUILD_DIR
# tar -xvf gnutls-3.6.16.tar.xz
# cd gnutls-3.6.16
# ./configure --prefix=$TARGET_DIR --disable-shared --enable-static --disable-doc --without-p11-kit
# make -j$(nproc)
# make install

```
