# Copyright (c) Jupyter Development Team.
# Distributed under the terms of the Modified BSD License.
FROM base-notebook-gpu

ARG ssh_prv_key
ARG ssh_pub_key

MAINTAINER aburnap@mit.edu
#MAINTAINER Jupyter Project <jupyter@googlegroups.com>

USER root

RUN echo "nameserver 8.8.8.8" > /etc/resolv.conf

# Install all OS dependencies for fully functional notebook server
RUN apt-get update && apt-get install -yq --no-install-recommends \
    build-essential \
    git \
    curl\
    freeglut3-dev \
    libcupti-dev \
    libcurl3-dev \
    libfreetype6-dev \
    libpng12-dev \
    libzmq3-dev \
    pkg-config \
    python-dev \
    rsync \
    software-properties-common \
    inkscape \
    jed \
    libsm6 \
    libxext-dev \
    libxrender1 \
    lmodern \
    pandoc \
    vim \
    unzip \
    libpng-dev \
    libc6-dbg \
    gdb \
    openssh-server \
    valgrind && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Authorize SSH Host
RUN mkdir -p ~/.ssh && \
    chmod 0700 ~/.ssh && \
    ssh-keyscan github.com > ~/.ssh/known_hosts

# Add the keys and set permissions
RUN echo "$ssh_prv_key" > ~/.ssh/id_rsa && \
    echo "$ssh_pub_key" > ~/.ssh/id_rsa.pub && \
    chmod 600 ~/.ssh/id_rsa && \
    chmod 600 ~/.ssh/id_rsa.pub

#ENV CUDNN_TAR_FILE cudnn-9.0-linux-x64-v7.0.tgz

##RUN wget http://developer.download.nvidia.com/compute/redist/cudnn/v7.0/$CUDNN_TAR_FILE && \
RUN wget https://s3.amazonaws.com/open-source-william-falcon/cudnn-9.0-linux-x64-v7.1.tgz && \
    #tar -xzvf $CUDNN_TAR_FILE && \
    tar -xzvf cudnn-9.0-linux-x64-v7.1.tgz && \
    cp -P cuda/include/cudnn.h /usr/local/cuda/include && \
    cp -P cuda/lib64/libcudnn* /usr/local/cuda/lib64/ && \
    chmod 777 /usr/local/cuda/lib64/libcudnn* && \
    rm -r cuda && \
    #rm $CUDNN_TAR_FILE
    rm cudnn-9.0-linux-x64-v7.1.tgz

# libav-tools for matplotlib anim
RUN apt-get update && \
    apt-get install -y --no-install-recommends libav-tools && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

RUN curl -O https://bootstrap.pypa.io/get-pip.py && \
    python get-pip.py && \
    rm get-pip.py

#COPY jupyter_notebook_config.py ~/.jupyter/

#RUN python -m pip3 install --upgrade pip

#RUN apt-get install -y python-setuptools

RUN pip3 --no-cache-dir install \
        Pillow \
        h5py \
        ipykernel \
        jupyter \
        matplotlib \
        seaborn \
        numpy \
        pandas \
        scipy \
        sklearn \
        && \
python -m ipykernel.kernelspec

RUN pip3 --no-cache-dir install \
        tensorflow \
        tensorflow-gpu \
        edward \
        keras && \
python -m ipykernel.kernelspec

RUN pip3 --no-cache-dir install \
        tensorboard \
        python-dateutil \
        easydict \
        torchfile \
        pandas && \
python -m ipykernel.kernelspec

#RUN conda install pytorch torchvision -c soumith && \
RUN conda install pytorch torchvision cuda90 -c pytorch && \
    conda clean -tipsy && \
    fix-permissions $CONDA_DIR

ENV CUDA_HOME /usr/local/cuda-9.0
ENV LD_LIBRARY_PATH /usr/local/cuda-9.0:/usr/local/cuda-9.0/lib64:$LD_LIBRARY_PATH
ENV LD_LIBRARY_PATH /usr/local/cuda-9.0/extras/CUPTI/lib64:$LD_LIBRARY_PATH
ENV LIBRARY_PATH $LD_LIBRARY_PATH

# Install facets which does not have a pip or conda package at the moment
RUN cd /tmp && \
    git clone https://github.com/PAIR-code/facets.git && \
    cd facets && \
    jupyter nbextension install facets-dist/ --sys-prefix && \
    rm -rf facets && \
    fix-permissions $CONDA_DIR

# Import matplotlib the first time to build the font cache.
ENV XDG_CACHE_HOME /home/$NB_USER/.cache/
RUN MPLBACKEND=Agg python -c "import matplotlib.pyplot" && \
    fix-permissions /home/$NB_USER

USER $NB_USER
