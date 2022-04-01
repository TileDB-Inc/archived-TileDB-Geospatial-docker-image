FROM ubuntu:20.04
LABEL maintainer="support@tiledb.io"

ENV DEBIAN_FRONTEND=noninteractive
ENV TZ=GMT
ENV LD_LIBRARY_PATH=/usr/local/lib:${LD_LIBRARY_PATH}
ENV LANG=C.UTF-8
ENV LC_ALL=C.UTF-8

RUN apt-get update && apt-get install -y \
  gosu \
  pwgen \
  tzdata \
  gcc \
  g++ \
  build-essential \
  sqlite \
  libsqlite3-dev \
  libxml2-dev \
  libjpeg-dev \
  libpng-dev \
  libfreetype6-dev \  
  libzstd-dev \
  libnetcdf-dev \
  python3-pip \
  git \
  wget \
  swig \
  zip \
  && rm -rf /var/lib/apt/lists/*

# Install latest cmake
RUN mkdir /build_deps && cd /build_deps \
  && wget --no-check-certificate https://cmake.org/files/LatestRelease/cmake-3.23.0-linux-x86_64.sh -O cmake-3.23.0-linux-x86_64.sh \
  && mkdir -p /opt/cmake \
  && sh cmake-3.23.0-linux-x86_64.sh --prefix=/opt/cmake --skip-license \
  && ln -s /opt/cmake/bin/cmake /usr/local/bin/cmake \
  && cmake --version

# Install tiledb using 2.6.4 release
RUN cd /build_deps \
  && git clone https://github.com/TileDB-Inc/TileDB.git -b 2.6.4 && cd TileDB \
  && mkdir build && cd build \
  && ../bootstrap --prefix=/usr/local --enable-s3 --enable-serialization \
  && make && make -C tiledb install

# Install curl after building tiledb
RUN apt-get update && apt-get install -y \
  libcurl4 \
  libcurl4-openssl-dev \
  && rm -rf /var/lib/apt/lists/*

# Install OpenJPEG
RUN cd /build_deps \
  && git clone https://github.com/uclouvain/openjpeg.git -b v2.2.0 && cd openjpeg \
  && mkdir build && cd build \
  && cmake .. \
  && make -j$(nproc) \
  && make install

# Install libtiff
RUN cd /build_deps \
  && wget --no-check-certificate https://download.osgeo.org/libtiff/tiff-4.1.0.tar.gz \
  && tar -zxf tiff-4.1.0.tar.gz \
  && cd tiff-4.1.0 \
  && ./configure \
  && make \
  && make install

# Install Proj
RUN cd /build_deps \
  && git clone https://github.com/OSGeo/PROJ.git -b 6.2.1 && cd PROJ \
  && mkdir build && cd build \
  && cmake .. \
  && make -j$(nproc) \
  && make install

# Install libgeotiff
RUN cd /build_deps \
  && wget --no-check-certificate https://download.osgeo.org/geotiff/libgeotiff/libgeotiff-1.5.1.tar.gz \
  && tar -zxf libgeotiff-1.5.1.tar.gz \
  && cd libgeotiff-1.5.1 \
  && mkdir build && cd build \
  && cmake .. \
  && make \
  && make install

# Install GDAL
RUN wget -q https://github.com/OSGeo/gdal/archive/refs/tags/v3.5.0alpha1.zip \
  && unzip -q v3.5.0alpha1.zip \
  && cd gdal-3.5.0alpha1 \
  && mkdir build \
  && cd build \
  && cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=$CONDA_DIR .. \
  && make -j$(nproc) \
  && make install \
  && cd /tmp \
  && rm -rf gdal-3.5.0alpha1 v3.5.0alpha1.zip

## Install TileDB-Py
RUN cd /build_deps \
  && pip3 install numpy \
  && git clone https://github.com/TileDB-Inc/TileDB-Py.git -b 0.13.1 \
  && cd TileDB-Py && python3 setup.py install

## Install XArray
RUN cd /build_deps && pip3 install xarray

## Install Dask
RUN cd /build_deps \
  && pip3 install toolz && pip3 install dask_image \
  && git clone https://github.com/dask/dask.git && cd dask \
  && git checkout 807f3225cf840f28ce7cf89b88fea63d473889e7 \
  && python3 setup.py install \
  && pip3 install dask distributed --upgrade \
  && pip3 install dask-image

# Install PDAL
RUN cd /build_deps \
  && git clone https://github.com/TileDB-Inc/PDAL.git && cd PDAL \
  && git checkout nb/tiledb_2_x \
  && mkdir build && cd build \
  && cmake .. -G "Unix Makefiles" .. -DBUILD_PLUGIN_TILEDB=ON \
  && make \
  && make install

# Install Rasterio, fiona, shapely and pdal client
RUN cd /build_deps && pip3 install cython \
  && pip3 install --no-cache-dir rasterio --no-binary=rasterio \
  fiona \
  shapely \
  pdal~=3.0

# Install TileDB-SAR
RUN cd /build_deps \
   && git clone https://github.com/TileDB-Inc/TileDB-SAR.git && cd TileDB-SAR \
   && git checkout 888059a15d87ae95fff6dc01c8bd4343ee4eaee1 \
   && python3 setup.py install

# Install Mapserver
RUN cd /build_deps \
  && git clone https://github.com/mapserver/mapserver.git && cd mapserver \
  && git checkout 0fcc810f0b559c800f950db78a79fa6574799f23 \
  && mkdir build && cd build \
  && cmake .. -DWITH_GIF=OFF -DWITH_HARFBUZZ=OFF -DWITH_PROTOBUFC=OFF -DWITH_FRIBIDI=OFF -DWITH_POSTGIS=OFF -DWITH_GEOS=OFF -DWITH_FCGI=OFF -DWITH_CAIRO=OFF \
  && make \
  && make install

# Clean up
RUN cd /tmp && rm -r /build_deps
