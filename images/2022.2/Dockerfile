FROM xilinx-ubuntu-20.04.4-user:v2022.2 AS build

# Install Miniforge
USER root
RUN curl -L -O "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-$(uname)-$(uname -m).sh" \
    && bash Miniforge3-$(uname)-$(uname -m).sh -b -p /opt/conda \
    && source "/opt/conda/etc/profile.d/conda.sh" \
    && echo "export PATH=/opt/conda/bin:$PATH" > /etc/profile.d/conda.sh \
USER runner
ENV PATH=/opt/conda/bin:$PATH
#setups stuff
RUN echo "source activate base" > ~/.bashrc
#install packages
COPY files/hls4ml.yml /tmp/hls4ml.yml
RUN conda env create --name hls4ml --file /tmp/hls4ml.yml

RUN conda install -y -c conda-forge conda-pack

RUN conda-pack -n acord-dac-cicd -o /tmp/env.tar && \
  mkdir /venv && cd /venv && tar xf /tmp/env.tar && \
  rm /tmp/env.tar

RUN /venv/bin/conda-unpack

FROM xilinx-ubuntu-20.04.4-user:v2022.2 AS deploy

LABEL org.opencontainers.image.source=https://github.com/ben-hawks/hls4ml-docker
LABEL org.opencontainers.image.path="images/2023.2/Dockerfile"
LABEL org.opencontainers.image.title="hls4ml-v1.0.0_2022.2"
LABEL org.opencontainers.image.version="v1.0.0_v2022.2"
LABEL org.opencontainers.image.description="A image for using hls4ml with Vitis 2022.2 and related tools."
LABEL org.opencontainers.image.authors="Ben Hawks (@ben-hawks)"
LABEL org.opencontainers.image.licenses="MIT"
LABEL org.opencontainers.image.documentation=https://github.com/ben-hawks/hls4ml-docker/README.md

# The UID env var should be used in child Containerfile.
ENV UID=1000
ENV GID=0
ENV USERNAME="xilinx"

# Vivado/Vitis Variables
ENV LIBRARY_PATH=/usr/lib/x86_64-linux-gnu
ENV XILINXD_LICENSE_FILE=2100@

COPY --from=build --chown=xilinx /venv /venv

WORKDIR /home/${USERNAME}

RUN echo "source /venv/bin/activate" >> ~/.bashrc
RUN echo "source /opt/Xilinx/Petalinux/2022.2/settings.sh" >> ~/.bashrc
RUN echo "source /opt/Xilinx/Vitis/2022.2/settings64.sh" >> ~/.bashrc
RUN echo "source /opt/Xilinx/Vivado/2022.2/settings64.sh" >> ~/.bashrc
RUN echo "export VIVADO=$(XILINX_VIVADO)" >> ~/.bashrc
RUN cat ~/.bashrc

RUN echo "source /venv/bin/activate" >> ~/.bashrc

RUN cat ~/.bashrc

# Add Tini - https://stackoverflow.com/questions/55733058/vivado-synthesis-hangs-in-docker-container-spawned-by-jenkins
ENV TINI_VERSION v0.19.0
ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini /tini
RUN sudo chmod +x /tini

SHELL ["/bin/bash", "-o", "pipefail", "-c", "-l"]
ENTRYPOINT ["/tini", "--" ]
CMD ["/bin/bash", "-o", "pipefail", "-c", "-l"]