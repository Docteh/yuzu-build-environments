FROM ubuntu:18.04
LABEL maintainer="yuzu"

ENV CLANG_VER=14
ENV CMAKE_VER=3.16.9
ENV DEBIAN_FRONTEND=noninteractive
ENV GCC_VER=11.3.0
ENV QT_PKG_VER=515
ENV QT_VER=5.15.2
ENV UBUNTU_VER=bionic

# Create a user account yuzu (UID 1027) that the container will run as
RUN useradd -m -u 1027 -s /bin/bash yuzu && \
    apt-get update && apt-get -y full-upgrade && \
    apt-get install --no-install-recommends -y \
    appstream \
    autoconf \
    automake \
    build-essential \
    ccache \
    desktop-file-utils \
    file \
    gpg-agent \
    libfile-mimeinfo-perl \
    libglu1-mesa-dev \
    libpulse-dev \
    libssl-dev \
    libtool \
    libudev-dev \
    libva-dev \
    libwayland-dev \
    libzip-dev \
    mesa-common-dev \
    nasm \
    ninja-build \
    patchelf \
    pkg-config \
    python3-pip \
    python3-setuptools \
    software-properties-common \
    unzip \
    wget \
    zlib1g-dev \
    zsync \
# vcpkg requirements
    curl \
    zip \
    && \
# Conan usage is deprecated
    pip3 install conan && \ 
# Install updated versions of glslang, git, and Qt from launchpad repositories
    add-apt-repository -y ppa:beineri/opt-qt-${QT_VER}-${UBUNTU_VER} && \
    add-apt-repository -y ppa:savoury1/graphics && \
    add-apt-repository -y ppa:savoury1/multimedia && \
    add-apt-repository -y ppa:savoury1/ffmpeg4 && \
    add-apt-repository -y ppa:ubuntu-toolchain-r/test && \
    add-apt-repository -y ppa:git-core/ppa && \
    apt-get update -y && \
    apt-get install --no-install-recommends -y \
    git \
    glslang-dev \
    glslang-tools \
    libhidapi-dev \
    qt${QT_PKG_VER}base \
    qt${QT_PKG_VER}tools \
    qt${QT_PKG_VER}wayland \
    qt${QT_PKG_VER}multimedia \
    qt${QT_PKG_VER}webengine && \
# Install clang from apt.llvm.org
    wget -O - https://apt.llvm.org/llvm-snapshot.gpg.key | apt-key add - && \
    echo "deb http://apt.llvm.org/${UBUNTU_VER}/ llvm-toolchain-${UBUNTU_VER}-${CLANG_VER} main" >> /etc/apt/sources.list && \
    apt-get update -y && \
    apt-get install --no-install-recommends -y \
    clang-${CLANG_VER} \
    lld-${CLANG_VER} \
    llvm-${CLANG_VER} \
    llvm-${CLANG_VER}-linker-tools && \
    ln -s $(which clang-${CLANG_VER}) /usr/bin/clang && \
    ln -s $(which clang++-${CLANG_VER}) /usr/bin/clang++ && \
    dpkg-reconfigure ccache && \
    apt-get clean autoclean && \
    apt-get autoremove --yes && \
    rm -rf /var/lib/apt /var/lib/dpkg /var/lib/cache /var/lib/log

# Install CMake from upstream
# yuzu requires CMake version 3.15, however Ubuntu only provides 3.10 to Bionic.
RUN cd /tmp && \
    wget --no-verbose https://github.com/Kitware/CMake/releases/download/v${CMAKE_VER}/cmake-${CMAKE_VER}-Linux-x86_64.tar.gz && \
    tar xvf cmake-${CMAKE_VER}-Linux-x86_64.tar.gz && \
    cp -rv cmake-${CMAKE_VER}-Linux-x86_64/* /usr && \
    rm -rf cmake-*

# Install Boost 1.79.0 from yuzu-emu/ext-linux-bin
RUN cd /tmp && \
    wget --no-verbose https://github.com/yuzu-emu/ext-linux-bin/raw/main/boost/boost-1_79_0.tar.xz && \
    tar xvf boost-1_79_0.tar.xz && \
    chown -R root:root boost-1_79_0/ && \
    cp -rv boost-1_79_0/usr / && \
    rm -rf boost*

# Install GCC from yuzu-emu/ext-linux-bin
RUN cd /tmp && \
    wget --no-verbose \
        https://github.com/yuzu-emu/ext-linux-bin/raw/main/gcc/gcc-${GCC_VER}-ubuntu.tar.xz.aa \
        https://github.com/yuzu-emu/ext-linux-bin/raw/main/gcc/gcc-${GCC_VER}-ubuntu.tar.xz.ab \
        https://github.com/yuzu-emu/ext-linux-bin/raw/main/gcc/gcc-${GCC_VER}-ubuntu.tar.xz.ac && \
    cat gcc-${GCC_VER}-ubuntu.tar.xz.* | tar xJ && \
    cp -rv gcc-${GCC_VER}/usr / && \
    rm -rf /tmp/gcc* && \
# Use updated libstdc++ and libgcc_s on the container from GCC 11
    rm -v /usr/lib/x86_64-linux-gnu/libstdc++.so.6 /lib/x86_64-linux-gnu/libgcc_s.so.1 && \
    ln -sv /usr/local/lib64/libstdc++.so.6.0.29 /usr/lib/x86_64-linux-gnu/libstdc++.so.6 && \
    ln -sv /usr/local/lib64/libgcc_s.so.1 /lib/x86_64-linux-gnu/libgcc_s.so.1 && \
# Help Clang find the updated GCC C++ version
    ln -sv /usr/local/include/c++/${GCC_VER}/ /usr/include/c++/${GCC_VER} && \
    ln -sv /usr/local/lib/gcc/x86_64-pc-linux-gnu/${GCC_VER} /usr/lib/gcc/x86_64-linux-gnu/${GCC_VER} && \
    cp -rv /usr/local/include/c++/${GCC_VER}/x86_64-pc-linux-gnu/* /usr/local/include/c++/${GCC_VER}/

# Setup paths for Qt binaries
ENV LD_LIBRARY_PATH=/opt/qt${QT_PKG_VER}/lib:${LD_LIBRARY_PATH}
ENV PATH=/opt/qt${QT_PKG_VER}/bin:${PATH}

# Fix GCC 11 <-> Qt 5.15 issue
COPY qtconcurrentthreadengine.patch /opt/qt515/qtconcurrentthreadengine.patch
RUN patch /opt/qt515/include/QtConcurrent/qtconcurrentthreadengine.h /opt/qt515/qtconcurrentthreadengine.patch && \
    rm /opt/qt515/qtconcurrentthreadengine.patch

# Install Catch2 manually in order to use older version than vcpkg provides
RUN cd /tmp && \
    git clone --depth 1 --branch v2.13.9 https://github.com/catchorg/Catch2.git && \
    cmake -B build -S Catch2 -G Ninja \
        -DCMAKE_INSTALL_PREFIX=/usr \
        -DCATCH_USE_VALGRIND=OFF \
        -DCATCH_BUILD_EXAMPLES=OFF \
        -DCATCH_ENABLE_COVERAGE=OFF \
        -DCATCH_ENABLE_WERROR=OFF \
        -DCATCH_BUILD_TESTING=ON && \
    ninja -C build install && \
    rm -rf build Catch2

# Tell CMake to use vcpkg when looking for packages
ENV VCPKG_TOOLCHAIN_FILE=/home/yuzu/vcpkg/scripts/buildsystems/vcpkg.cmake

USER 1027
# Conan usage is deprecated
COPY --chown=yuzu:yuzu settings.yml /home/yuzu/.conan/settings.yml
# Install vcpkg and required dependencies for yuzu
RUN cd /home/yuzu &&\
    git clone --depth 1 https://github.com/Microsoft/vcpkg.git &&\
    cd vcpkg &&\
    ./bootstrap-vcpkg.sh &&\
    ./vcpkg install \
        fmt \
        lz4 \
        nlohmann-json \
        zlib \
        zstd
