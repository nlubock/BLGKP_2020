# Start with rocker/tidyverse base image
FROM rocker/verse:3.6.2

#===============================================================================
# Fix umask so that everything run at rstudio/user level is group-writable
RUN echo "umask 0002" >> /home/rstudio/.bashrc

#===============================================================================
# Environment Variables
ENV starcode_version 1.3
ENV mlr_version 5.7.0

#===============================================================================
# Install extra *nix utils
# x11, mesa, glu1 are so we can install paletteer
RUN apt-get update \
    && apt-get install -y \
    pigz \
    vim \
    git \
    less \
    curl \
    wget \
    parallel

#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# Python Env
# Install miniconda to /miniconda
ENV LANG=C.UTF-8 LC_ALL=C.UTF-8
ENV PATH /opt/conda/bin:$PATH

RUN apt-get update --fix-missing \
    && apt-get install -y \
    bzip2 \
    ca-certificates \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

RUN wget --quiet https://repo.anaconda.com/miniconda/Miniconda3-4.5.11-Linux-x86_64.sh -O ~/miniconda.sh \
    && /bin/bash ~/miniconda.sh -b -p /opt/conda \
    && rm ~/miniconda.sh \
    && /opt/conda/bin/conda clean -tipsy \
    && ln -s /opt/conda/etc/profile.d/conda.sh /etc/profile.d/conda.sh \
    && echo ". /opt/conda/etc/profile.d/conda.sh" >> ~/.bashrc \
    && echo "conda activate base" >> ~/.bashrc \
    && echo ". /opt/conda/etc/profile.d/conda.sh" >> /home/rstudio/.bashrc \
    && echo "conda activate base" >> /home/rstudio/.bashrc

WORKDIR /opt/conda
RUN conda update -y conda \
    && conda install -y \
    -c conda-forge \
    numpy \
    seaborn \
    pandas \
    scikit-learn \
    ipython \
    jupyterlab \
    python-levenshtein \
    openpyxl \
    xlrd \
    anndata

#===============================================================================
# other utils

# bcl2fastq - derived from: https://github.com/Zymo-Research/docker-bcl2fastq
# ln is required to avoid some missing stat.h error
WORKDIR /tmp
RUN mkdir -p /usr/include/sys \
    && ln -s /usr/include/x86_64-linux-gnu/sys/stat.h /usr/include/sys/stat.h \
    && wget ftp://webdata2:webdata2@ussd-ftp.illumina.com/downloads/software/bcl2fastq/bcl2fastq2-v2-20-0-tar.zip \
    && unzip bcl2fastq2-v2-20-0-tar.zip \
    && tar xzvf bcl2fastq2-v2.20.0.422-Source.tar.gz \
    && ./bcl2fastq/src/configure --prefix=/usr/local/ \
    && make -j8 \
    && make install \
    && rm -r /tmp/*

#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# starcode
WORKDIR /usr/local
RUN set -u; \
    wget -O starcode.tar.gz "https://github.com/gui11aume/starcode/archive/${starcode_version}.tar.gz" \
    && tar xzf starcode.tar.gz \
    && cd "starcode-${starcode_version}" \
    && make \
    && cp starcode /usr/local/bin/starcode \
    && rm ../starcode.tar.gz

#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# mlr
WORKDIR /usr/local/bin
RUN set -u; \
    wget "https://github.com/johnkerl/miller/releases/download/v${mlr_version}/mlr.linux.x86_64" \
    && chmod 0777 mlr.linux.x86_64 \
    && mv mlr.linux.x86_64 mlr

#===============================================================================
# final touches

RUN echo "alias ll='ls -Alrth'" >> ~/.bashrc
RUN echo "alias ll='ls -Alrth'" >> /home/rstudio/.bashrc
WORKDIR /home/rstudio
