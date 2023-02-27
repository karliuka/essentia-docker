FROM ubuntu:18.04

ENV DEBIAN_FRONTEND=noninteractive

ENV LANG C.UTF-8

# tensorflow - the release for CPU and GPU
# tensorflow-cpu - the release for CPU-only
# tf-nightly - the preview build (unstable)
ENV TENSORFLOW_TYPE tensorflow-cpu
# tensorflow release version https://www.tensorflow.org/install/source#tested_build_configurations
ENV TENSORFLOW_VERSION 2.5.2

# install common dependencies and utils
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    python3-dev \
    python3-pip \
    python3-numpy-dev \
    python3-yaml \
    python3-six \
    libfftw3-3 \
    libyaml-0-2 \
    libtag1v5 \
    libsamplerate0 \
    libavcodec57 \
    libavformat57 \
    libavutil55 \
    libavresample3 \
    pkg-config \
    libeigen3-dev \
    wget \
    git

RUN rm -rf /var/lib/apt/lists/*

# install and upgrade pip
RUN python3 -m pip install --upgrade pip setuptools

# install tensorflow and dependencies
RUN pip3 install ${TENSORFLOW_TYPE}==${TENSORFLOW_VERSION}

# install gaia dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    libqt4-dev \
    libyaml-dev \
    qt5-default \
    swig

# load gaia
RUN mkdir /essentia && mkdir -p /usr/local/lib/pkgconfig \
    && cd /essentia && git clone https://github.com/MTG/gaia.git

# compile and install gaia
RUN cd /essentia/gaia \
    && python3 waf configure \
    && python3 waf && python3 waf install

# install essentia dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    libchromaprint-dev \
    libfftw3-dev \
    libavcodec-dev \
    libavformat-dev \
    libavresample-dev \
    libsamplerate0-dev \
    libtag1-dev

# load essentia
RUN cd /essentia && git clone https://github.com/MTG/essentia.git

# prepare tensorflow from essentia
RUN cd /essentia/essentia \
    && src/3rdparty/tensorflow/setup_from_python.sh \
    && echo "Requires: python3" >> /usr/local/lib/pkgconfig/tensorflow.pc

# compile and install essentia
RUN cd /essentia/essentia \
    && python3 waf configure \
    --build-static --with-python --with-examples --with-vamp --with-tensorflow --with-gaia \
    && python3 waf && python3 waf install && ldconfig

# install music tag
RUN pip3 install music-tag

# remove dependencies
RUN apt-get remove -y \
    build-essential \
    libyaml-dev \
    libfftw3-dev \
    libavcodec-dev \
    libavformat-dev \
    libavutil-dev \
    libavresample-dev \
    libsamplerate0-dev \
    libtag1-dev \
    python3-numpy-dev

# clean up
RUN apt-get autoremove -y && apt-get clean -y

# clean up filesystem
RUN rm -rf /var/lib/apt/lists/* \
    && cd / && rm -rf /essentia

ENV PYTHONPATH /usr/local/lib/python3/dist-packages

WORKDIR /essentia
