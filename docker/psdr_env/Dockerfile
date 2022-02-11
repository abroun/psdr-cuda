FROM nvidia/cuda:11.0-devel-ubuntu20.04
ARG USER_ID
ARG USER_NAME

## Install dependencies
RUN \
  apt-get update && \
  DEBIAN_FRONTEND=noninteractive apt-get install -y \
    python3 python3-dev python3-distutils python3-setuptools libopenexr-dev \
    cmake build-essential cmake-curses-gui git python3-pip

# Build and install Enoki
RUN mkdir /deps
WORKDIR /deps
RUN git clone --recursive https://github.com/mitsuba-renderer/enoki
WORKDIR /deps/enoki
#RUN git checkout 5754a0e4118d9c236ee3614cd9e5b8f9120f7af7
#RUN git submodule update --recursive
RUN mkdir /deps/enoki/build
WORKDIR /deps/enoki/build
#-D ENOKI_CUDA_COMPUTE_CAPABILITY=86
RUN cmake -D ENOKI_CUDA=ON \
    -D ENOKI_PYTHON=ON -D ENOKI_AUTODIFF=ON ../
RUN make -j
#RUN make install   # No install setup for Enoki... :P

# Build and install pybind11
WORKDIR /deps
RUN git clone https://github.com/pybind/pybind11.git
WORKDIR /deps/pybind11
RUN mkdir /deps/pybind11/build
WORKDIR /deps/pybind11/build
RUN cmake ../
RUN make -j
RUN make install

WORKDIR /usr/local
COPY deps/NVIDIA-OptiX-SDK-7.1.0-linux64-x86_64.sh .
RUN chmod a+x ./NVIDIA-OptiX-SDK-7.1.0-linux64-x86_64.sh
RUN ./NVIDIA-OptiX-SDK-7.1.0-linux64-x86_64.sh --skip-license --include-subdir

RUN apt-get update && apt-get install -y xterm

RUN apt-get update && apt-get install -y libglib2.0-0 python3-opencv
RUN pip install numpy 
#opencv-python

WORKDIR /src

# Setup username and id so that it matches host machine (otherwise files will be written as root)
RUN useradd -u ${USER_ID} ${USER_NAME}
ENV USER=${USER_NAME}
ENV USER=${USER_NAME}
ENV HOME=/home/${USER_NAME}
RUN mkdir -p /home/${USER_NAME}
RUN chown ${USER_NAME}:${USER_NAME} /home/${USER_NAME}
USER ${USER_NAME}

ENV LD_LIBRARY_PATH=/usr/local/nvidia/lib:/usr/local/nvidia/lib64:/deps/enoki/build
ENV PYTHONPATH=/src/build/lib:/deps/enoki/build


# Inside container build psdr_cuda using the following
# mkdir build
# cd build
# ccmake -D PYTHON_INCLUDE_PATH=/usr/include/python3.8/ -D OpenEXR_ROOT=/usr/include/OpenEXR/ -D ENOKI_DIR=/deps/enoki/ ../
    
