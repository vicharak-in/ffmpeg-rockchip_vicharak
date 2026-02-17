Building a Debian Package for ffmpeg
------------------------------------

# Get the Dependencies

These are packages required for compiling. These can removed later if not required.

```
sudo apt update && sudo apt -y install \
  autoconf \
  automake \
  build-essential \
  cmake \
  debhelper \
  git-core \
  libass-dev \
  libfreetype6-dev \
  libgnutls28-dev \
  libmp3lame-dev \
  libsdl2-dev \
  libtool \
  libva-dev \
  libvdpau-dev \
  libvorbis-dev \
  libxcb1-dev \
  libxcb-shm0-dev \
  libxcb-xfixes0-dev \
  meson \
  ninja-build \
  pkg-config \
  texinfo \
  wget \
  yasm \
  zlib1g-dev \
  git \
  gcc \
  libdrm-dev
```

In your home directory make a new directory to pull of the source code into:
```
mkdir -p ~/ffmpeg-dev
```

# Compilation and Installation
This guide assumes that that you want to install some of the most common third-party libraries. If you do not require certain features, do not install the library and remove the appropriate ``./configure`` option in FFmpeg. For example, if libvpx is not required, do not install libvpx-dev and then remove ``--enable-libvpx``from the ``./configure`` option from ``debian/rules``.

## NASM
An assembler used by some libraries. Install it using the following command:
```
sudo apt-get install nasm
```

## Build RKMPP (Rockchip Media Process Platform)
This is the MPP comaptible with Rockchip platforms. It includes support for hardware encoders and decoders. Required to configure ffmpeg with ``--enable-rkmpp`` option.
```
mkdir -p ~/ffmpeg-dev && cd ~/ffmpeg-dev 
git clone -b jellyfin-mpp --depth=1 https://github.com/nyanmisaka/mpp.git rkmpp
pushd rkmpp
mkdir rkmpp_build
pushd rkmpp_build
cmake \
    -DCMAKE_INSTALL_PREFIX=/usr \
    -DCMAKE_BUILD_TYPE=Release \
    -DBUILD_SHARED_LIBS=ON \
    -DBUILD_TEST=OFF \
    ..
make -j $(nproc)
sudo make install
```

If the above nyanmisaka's repo doesn't work out, use the below one. Generally below repo works for
all chips. Nyanmisaka's was used for rk3588

```
cd ~/ffmpeg-dev
rm -rf rkmpp

git clone https://github.com/HermanChen/mpp.git
cd mpp

git checkout release
mkdir -p build && cd build/linux
cmake \
  -DCMAKE_INSTALL_PREFIX=/usr \
  -DCMAKE_BUILD_TYPE=Release \
  -DBUILD_SHARED_LIBS=ON \
  -DBUILD_TEST=OFF \
  ..
make -j$(nproc)
sudo make install
```

## Build RKRGA (Rockchip Raster Graphic Acceleration)
This is the RGA version compatible with Rockchip platforms. It includes support for hardware filters. Required to configure ffmpeg with ``--enable-rkrga`` option.
```
mkdir -p ~/ffmpeg-dev && cd ~/ffmpeg-dev
git clone -b jellyfin-rga --depth=1 https://github.com/nyanmisaka/rk-mirrors.git rkrga
meson setup rkrga rkrga_build \
    --prefix=/usr \
    --libdir=lib \
    --buildtype=release \
    --default-library=shared \
    -Dcpp_args=-fpermissive \
    -Dlibdrm=false \
    -Dlibrga_demo=false
PAGER=cat meson configure rkrga_build
sudo ninja -C rkrga_build install
```

## libx264
Provides H.264 software video encoder. Required to configure ffmpeg with ``--enable-libx264`` option.
```
sudo apt-get install libx264-dev
```

## libx265
Provides H.265 software video encoder. Required to configure ffmpeg with ``--enable-libx265`` option.
```
mkdir -p ~/ffmpeg-dev && cd ~/ffmpeg-dev
wget -O x265.tar.bz2 https://bitbucket.org/multicoreware/x265_git/get/master.tar.bz2 && \
tar xjvf x265.tar.bz2 && \
cd multicoreware*/build/linux && \
cmake -G "Unix Makefiles" \
  -DCMAKE_INSTALL_PREFIX=/usr \
  -DCMAKE_BUILD_TYPE=Release \
  -DENABLE_SHARED=ON \
  -DENABLE_STATIC=ON \
  -DENABLE_CLI=ON \
  -DENABLE_SVE2=OFF \
  -DENABLE_SVE=OFF \
  -DCROSS_COMPILE_ARM64=OFF \
  ../../source
make
sudo make install
```

## libvpx
Provides VP8/VP9 software video encoder/decoder. Required to configure ffmpeg with ``--enable-libvpx`` option.
```
sudo apt-get install libvpx-dev
```

## libfdk-aac
Provides AAC audio encoder. Required to configure ffmpeg with ``--enable-libvpx`` (and ``--enable-nonfree`` if you also included ``--enable-gpl``).
```
sudo apt-get install libfdk-aac-dev
```

## libopus
Provides Opus audio decoder and encoder. Required to configure ffmpeg with ``--enable-libopus``
```
sudo apt-get install libopus-dev
```

## libaom
Provides AV1 software video encoder/decoder. Required to configure ffmpeg with ``--enable-libaom`` option.
```
mkdir -p ~/ffmpeg-dev && cd ~/ffmpeg-dev
git -C aom pull 2>/dev/null || git clone --depth=1 https://aomedia.googlesource.com/aom
mkdir -p aom_build
cmake -S aom -B aom_build \
  -DCMAKE_INSTALL_PREFIX=/usr \
  -DCMAKE_BUILD_TYPE=Release \
  -DENABLE_SHARED=ON \
  -DENABLE_STATIC=ON \
  -DENABLE_TESTS=OFF \
  -DENABLE_NASM=ON
cmake -L aom_build | grep ENABLE
cmake --build aom_build -j$(nproc)
sudo cmake --install aom_build
```

## libsvtav1
Provides AV1 software video encoder/decoder. Only the encoder is supported by FFmpeg, so building of the decoder is disabled. Required to configure ffmpeg with ``--enable-libsvtav1`` option.
```
mkdir -p ~/ffmpeg-dev && cd ~/ffmpeg-dev
git -C SVT-AV1 pull 2>/dev/null || git clone --depth=1 https://gitlab.com/AOMediaCodec/SVT-AV1.git
cd SVT-AV1
git fetch --tags
git checkout v1.5.0
cd ..
mkdir -p SVT-AV1/build
cmake -S SVT-AV1 -B SVT-AV1/build -DCMAKE_INSTALL_PREFIX=/usr -DCMAKE_BUILD_TYPE=Release -DBUILD_DEC=OFF -DBUILD_ENC=ON -DBUILD_SHARED_LIBS=ON
cmake -L SVT-AV1/build | grep BUILD_
cmake --build SVT-AV1/build
sudo cmake --install SVT-AV1/build
```

## libdav1d
Provides AV1 software decoder which is much faster than the one provided by libaom. Required to configure ffmpeg with ``--enable-libdav1d`` option.
```
sudo apt-get install libdav1d-dev
```

## libvmaf
It is a library for calculating the VMAF video quality metric. Required to configure ffmpeg with ``--enable-libvmaf`` option.
```
mkdir -p ~/ffmpeg-dev && cd ~/ffmpeg-dev
git -C vmaf pull 2>/dev/null || git clone https://github.com/Netflix/vmaf vmaf
mkdir -p vmaf/libvmaf/build
meson setup vmaf/libvmaf/build vmaf/libvmaf \
  --prefix=/usr \
  --libdir=lib \
  --buildtype=release \
  --default-library=shared \
  -Denable_tests=false \
  -Denable_docs=false
ninja -C vmaf/libvmaf/build
sudo ninja -C vmaf/libvmaf/build install
```

## libflite
It is a library providing speech synthesis (text-to-speech) functionality. Required to configure FFmpeg with the ``--enable-libflite`` option.
```
mkdir -p ~/ffmpeg-dev && cd ~/ffmpeg-dev
git clone https://github.com/festvox/flite.git
cd flite
./configure
make
make get_voices
sudo make install
```

Check if this library exists ``/usr/lib/aarch64-linux-gnu/libflite.so``. If not do the following steps:
```
cd /usr/lib/aarch64-linux-gnu/
sudo ln -sf /usr/lib/aarch64-linux-gnu/libflite.so.1 /usr/lib/aarch64-linux-gnu/libflite.so
cd ~/ffmpeg-dev
```

## Installing other libraries
These are some libraries which you can install enable different options while configuring ffpmeg. If you do not want a specific feature then dont install the library and remove specific option from the ``./configure`` command in the file ``debian/rules``.
```
sudo apt install libopenal-dev ocl-icd-opencl-dev opencl-headers libopengl-dev libomxil-bellagio-dev liblilv-dev libwebp-dev libxvidcore-dev libzimg-dev libzvbi-dev libxml2-dev libshine-dev libsnappy-dev libsoxr-dev libssh-dev libspeex-dev libsrt-gnutls-dev libtheora-dev libtwolame-dev libvidstab-dev librubberband-dev librabbitmq-dev libopenmpt-dev libmysofa-dev libjack-dev libgsm1-dev libgme-dev libfribidi-dev libfontconfig-dev libcodec2-dev libcdio-dev libcdio-paranoia-dev libcaca-dev libbs2b-dev libbluray-dev ladspa-sdk libc6-dev libopenjp2-7-dev libzmq3-dev libflite1
```

## Compile ffmpeg and build a custom debian package
Clone the ``debian`` branch ffmpeg-rockchip\_vicharak:
```
git clone -b vicharak-jammy https://github.com/vicharak-in/ffmpeg-rockchip_vicharak
cd ffmpeg-rockchip_vicharak
```

Build the debian package
```
dpkg-buildpackage -b -us -uc -nc -Zxz
```
Thus a debain package will be created in the parent directory.
