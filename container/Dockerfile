FROM heroku/heroku:20-build

ENV DEBIAN_FRONTEND=noninteractive

ARG COMPILE_VERSION_LIBDE265=1.0.15
ARG COMPILE_VERSION_LIBAOM=3.10.0
ARG COMPILE_VERSION_LIBHEIF=1.18.2
ARG COMPILE_VERSION_LIBWEBP=1.4.0
ARG COMPILE_VERSION_IMAGEMAGICK=6.9.12-98

RUN apt-get -yq update \
  # Needed for libaom library cmake optimizations
  && apt-get install -yq yasm cmake \
  # Install the latest libde265 library
  && curl -sSL --retry 3 https://github.com/strukturag/libde265/releases/download/v$COMPILE_VERSION_LIBDE265/libde265-$COMPILE_VERSION_LIBDE265.tar.gz | tar zx \
  && cd libde265-$COMPILE_VERSION_LIBDE265 \
  && ./autogen.sh \
  && ./configure \
  && make -j$(nproc) \
  && make install \
  && ldconfig /usr/local/lib \
  && cd ..
  # Install the latest libwebp library
RUN curl -sSL --retry 3 http://storage.googleapis.com/downloads.webmproject.org/releases/webp/libwebp-$COMPILE_VERSION_LIBWEBP.tar.gz | tar zx \
  && cd libwebp-$COMPILE_VERSION_LIBWEBP \
  && ./autogen.sh \
  && ./configure --enable-shared --enable-libwebpdecoder --enable-libwebpdemux --enable-libwebpmux --enable-libsharpyuv --enable-static=no \
  && make -j$(nproc) \
  && make install \
  && ldconfig /usr/local/lib \
  && cd ..
  # Install the latest libaom library
RUN git clone -b v${COMPILE_VERSION_LIBAOM} --depth 1 https://aomedia.googlesource.com/aom \
  && mkdir build_aom \
  && cd build_aom \
  && cmake ../aom/ -DENABLE_TESTS=0 -DBUILD_SHARED_LIBS=1 \
  && make -j$(nproc) \
  && make install \
  && ldconfig /usr/local/lib \
  && cd ..
  # Install the latest libheif library
RUN curl -sSL --retry 3 https://github.com/strukturag/libheif/releases/download/v$COMPILE_VERSION_LIBHEIF/libheif-$COMPILE_VERSION_LIBHEIF.tar.gz | tar zx \
  && cd libheif-$COMPILE_VERSION_LIBHEIF \
  && mkdir build \
  && cd build \
  && cmake --preset=release-noplugins .. \
  && make -j$(nproc) \
  && make install \
  && ldconfig /usr/local/lib \
  && cd ../
  # Get, configure and install ImageMagick
RUN curl -sSL --retry 3 https://github.com/ImageMagick/ImageMagick6/archive/refs/tags/$COMPILE_VERSION_IMAGEMAGICK.tar.gz | tar zx \
  && cd ImageMagick6-$COMPILE_VERSION_IMAGEMAGICK \
  && ./configure --prefix=/usr/src/imagemagick --disable-docs --with-avif=yes --with-heic=yes --with-webp=yes --with-tiff=yes --with-openexr=no --with-x=no \
  && make -j$(nproc) \
  && make install \
  && ldconfig /usr/local/lib \
  && cd ..
  # Copy shared libraries just compiled
RUN cp /usr/local/lib/libaom.so.3 /usr/src/imagemagick/lib \
  && cp /usr/local/lib/libde265.so.0 /usr/src/imagemagick/lib \
  && cp /usr/local/lib/libheif.so.1 /usr/src/imagemagick/lib \
  && cp /usr/local/lib/libwebp.so.7 /usr/src/imagemagick/lib \
  && cp /usr/local/lib/libsharpyuv.so.0 /usr/src/imagemagick/lib \

  # Clean up the build and get ready for packaging
  && cd /usr/src/imagemagick \
  && strip lib/*.a lib/lib*.so* \
  # Wrap it up with a bow (compress the binary)
  && rm -rf build \
  && mkdir build \
  && tar cjf /usr/src/imagemagick/build/imagemagick.tar.bz2 bin include lib etc share
