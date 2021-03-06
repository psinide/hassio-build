FROM %%BASE_IMAGE%%

# Add env
ENV LANG C.UTF-8

####
# Install core
RUN apk add --no-cache \
        python3 \
        bsd-compat-headers \
        libgcc \
        libstdc++ \
        libpng \
        tiff \
        libjpeg \
        libjpeg-turbo \
        libressl \
        musl \
    && ln -s /usr/include/locale.h /usr/include/xlocale.h

####
## Build library
WORKDIR /usr/src/

# ssocr
RUN apk add --no-cache git gcc make musl-dev imlib2-dev imlib2 \
    && git clone --depth 1 https://github.com/auerswal/ssocr \
    && cd ssocr \
    && make \
    && make install \
    && apk del git gcc make imlib2-dev musl-dev \
    && rm -rf /usr/src/ssocr

# OpenCV/scipy
ENV PYTHONPATH /usr/local/lib/python3.6/site-packages
RUN apk add --no-cache \
        gcc g++ musl-dev python3-dev cmake make ncurses linux-headers \
        lapack-dev lapack \
        gfortran libgfortran \
        ffmpeg-dev ffmpeg-libs \
        libwebp-dev libwebp \
        git libpng-dev libjpeg-turbo-dev tiff-dev \
    && pip3 install --no-cache-dir numpy scipy \
    && git clone --depth 1 -b 3.2.0 https://github.com/opencv/opencv \
    && cd opencv \
    && mkdir -p build \
    && cd build \
    && cmake ../ -DBUILD_TESTS=OFF -DBUILD_EXAMPLES=OFF -DBUILD_DOCS=OFF -DBUILD_PERF_TESTS=OFF -DBUILD_TESTS=OFF \
    && make \
    && make install \
    && apk del \
        gcc g++ musl-dev python3-dev lapack-dev linux-headers \
        gfortran cmake make ncurses ffmpeg-dev \
        git libpng-dev libjpeg-turbo-dev tiff-dev libwebp-dev \
    && rm -rf /usr/src/opencv
    
# Dlib
RUN apk add --no-cache \
        boost boost-python3 boost-dev \
        cmake ncurses make g++ python3-dev \
    && pip3 install --no-cache-dir dlib \
    && apk del boost-dev cmake ncurses make g++ python3-dev

# OpenALPR
RUN apk add --no-cache \
        cmake ncurses make g++ tesseract-ocr-dev tesseract-ocr \
        git autoconf automake \
    && git clone --depth 1 -b REL_1_2_0 https://github.com/log4cplus/log4cplus \
    && cd log4cplus \
    && ./configure \
    && sed -i "s/1\.14/1.15/g" aclocal.m4 \
    && sed -i "s/1\.14/1.15/g" Makefile.in \
    && make \
    && make install \
    && rm -rf /usr/src/log4cplus
RUN git clone --depth 1 -b v2.3.0 https://github.com/openalpr/openalpr \
    && cd openalpr/src \
    && mkdir -p build \
    && cd build \
    && cmake ../ -DWITH_TESTS=FALSE -DWITH_BINDING_JAVA=FALSE -DWITH_BINDING_PYTHON=FALSE -DWITH_BINDING_GO=FALSE -DWITH_DAEMON=FALSE -DCMAKE_INSTALL_PREFIX:PATH=/usr/local \
    && make \
    && make install \
    && apk del \
        cmake ncurses make g++ tesseract-ocr-dev \
        git autoconf automake \
    && rm -rf /usr/src/openalpr

# libcoap
RUN apk add --no-cache \
        git make gcc g++ autoconf automake libtool \
    && git clone --depth 1 --recursive -b dtls https://github.com/home-assistant/libcoap \
    && cd libcoap \
    && grep -r -l u_int8_t * | xargs sed -i "s/u_int8_t/unsigned char/g" \
    && grep -r -l u_int32_t * | xargs sed -i "s/u_int32_t/unsigned int/g" \
    && grep -r -l u_int64_t * | xargs sed -i "s/u_int64_t/unsigned long long/g" \
    && grep -r -l "time_t coap_time_t" * | xargs sed -i "s/time_t coap_time_t/unsigned long long coap_time_t/g" \
    && ./autogen.sh \
    && ./configure --disable-documentation --disable-shared --without-debug CFLAGS="-D COAP_DEBUG_FD=stderr" \
    && make \
    && make install \
    && apk del \
        git make gcc g++ autoconf automake libtool \
    && rm -rf /usr/src/libcoap

##
# Install component packages
RUN apk add --no-cache \
    ffmpeg \
    nmap \
    libcec \
    net-tools \
    bluez \
    curl \
    libsodium \
    openssh-client \
    libffi \
    glib

####
## Install pip module for component/homeassistant
RUN apk add --no-cache \
        gcc g++ musl-dev python3-dev make \
        mariadb-dev mariadb-client-libs \
        postgresql-dev postgresql-libs \
        freetds-dev freetds \
    && pip3 install --no-cache-dir mysqlclient psycopg2 \
    && pip3 install --no-cache-dir https://github.com/wayfair/pymssql/archive/v2.1.3.0.0.1.zip \
    && pip3 install --no-cache-dir cchardet uvloop aiodns \
    && apk del \
        mariadb-dev postgresql-dev freetds-dev \
        gcc g++ musl-dev python3-dev make

