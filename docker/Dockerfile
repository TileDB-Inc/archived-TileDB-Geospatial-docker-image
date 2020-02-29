FROM ubuntu:18.04
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
  cmake \
  sqlite \
  libsqlite3-dev \
  libxml2-dev \
  libjpeg-dev \
  libpng-dev \
  libfreetype6-dev \  
  libzstd-dev \
  python3-pip \
  git \
  wget \
  && rm -rf /var/lib/apt/lists/*

# Install tiledb using 1.7.5 release
RUN mkdir -p /build_deps && cd /build_deps \
  && git clone https://github.com/TileDB-Inc/TileDB.git -b 1.7.5 && cd TileDB \
  && mkdir -p build && cd build \
  && cmake -DTILEDB_VERBOSE=ON -DTILEDB_S3=ON -DTILEDB_SERIALIZATION=ON -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr/local .. \
  && make -j$(nproc) \
  && make -C tiledb install

# Install curl after building tiledb
RUN apt-get update && apt-get install -y \
  libcurl4 \
  libcurl4-openssl-dev \
  && rm -rf /var/lib/apt/lists/*

# Install OpenJPEG
RUN cd /build_deps \
  && git clone https://github.com/uclouvain/openjpeg.git -b v2.2.0 && cd openjpeg \
  && mkdir -p build && cd build \
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
  && mkdir -p build && cd build \
  && cmake .. \
  && make -j$(nproc) \
  && make install

# Install libgeotiff
RUN cd /build_deps \
  && wget --no-check-certificate https://download.osgeo.org/geotiff/libgeotiff/libgeotiff-1.5.1.tar.gz \
  && tar -zxf libgeotiff-1.5.1.tar.gz \
  && cd libgeotiff-1.5.1 \
  && mkdir -p build && cd build \
  && cmake .. \
  && make \
  && make install

# Install GDAL
RUN cd /build_deps \
  && git clone https://github.com/OSGeo/gdal.git && cd gdal/gdal \
  && git checkout 8385b711eef5ed9d034f215e8fa3507c90cd3471 \
  && ./configure --with-crypto=no --with-curl=no \
  && make -j$(nproc) \
  && make install

## Install TileDB-Py
RUN cd /build_deps \
  && pip3 install numpy \
  && git clone https://github.com/TileDB-Inc/TileDB-Py.git -b 0.5.6 \
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

# Install Rasterio
RUN cd /build_deps && pip3 install cython
RUN cd /build_deps \
  && git clone https://github.com/mapbox/rasterio.git && cd rasterio \
  && git checkout 74bd625dfd984ca090a1ee02f4fc1940b11d7ce8 \
  && python3 setup.py install

# Install Fiona
RUN cd /build_deps \
  && git clone https://github.com/Toblerity/Fiona.git && cd Fiona \
  && python3 setup.py install

# Install TileDB-SAR
RUN cd /build_deps \
   && git clone https://github.com/TileDB-Inc/TileDB-SAR.git && cd TileDB-SAR \
   && git checkout 888059a15d87ae95fff6dc01c8bd4343ee4eaee1 \
   && python3 setup.py install

# Install Mapserver
RUN cd /build_deps \
  && git clone https://github.com/mapserver/mapserver.git && cd mapserver \
  && git checkout 0fcc810f0b559c800f950db78a79fa6574799f23 \
  && mkdir -p build && cd build \
  && cmake .. -DWITH_GIF=OFF -DWITH_HARFBUZZ=OFF -DWITH_PROTOBUFC=OFF -DWITH_FRIBIDI=OFF -DWITH_POSTGIS=OFF -DWITH_GEOS=OFF -DWITH_FCGI=OFF -DWITH_CAIRO=OFF \
  && make \
  && make install

# Install LasZIP
RUN cd /build_deps \
  && wget https://github.com/LASzip/LASzip/releases/download/3.4.1/laszip-src-3.4.1.tar.gz \
  && tar -zxf laszip-src-3.4.1.tar.gz \
  && cd laszip-src-3.4.1 \
  && mkdir -p build && cd build \
  && cmake .. \
  && make \
  && make install

# Install PDAL
RUN cd /build_deps \
  && git clone https://github.com/PDAL/PDAL.git && cd PDAL \
  && git checkout 5be7c48b3501a6092aef70fc35ee9d75f11a61f4 \
  && mkdir -p build && cd build \
  && cmake .. \
  && make \
  && make install

# Install PDAL Python
RUN pip3 install packaging \
  && git clone https://github.com/PDAL/python pdalextension \
  && cd pdalextension \
  && python3 setup.py build \
  && python3 setup.py install

# Clean up
RUN cd /tmp && rm -r /build_deps
