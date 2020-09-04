FROM k8scc01covidacr.azurecr.io/remote-desktop-test:vscode-hotfix-base

USER root

# RStudio Server (In-Browser)
ARG RSTUDIO_VERSION=1.1.463
ARG SHA256=62aafd46f79705ca5db9c629ce3b60bf708d81c06a6b86cc4b417fbaf30691c1
RUN apt-get update && \
    wget http://archive.ubuntu.com/ubuntu/pool/main/o/openssl1.0/libssl1.0.0_1.0.2n-1ubuntu5.3_amd64.deb -O libssl1.0.0.deb && \
    dpkg -i libssl1.0.0.deb && \
    curl --silent -L --fail "https://download2.rstudio.org/rstudio-server-${RSTUDIO_VERSION}-amd64.deb" > /tmp/rstudio.deb && \
    echo "${SHA256} /tmp/rstudio.deb" | sha256sum -c - && \
    apt-get install --no-install-recommends -y /tmp/rstudio.deb && \
    rm /tmp/rstudio.deb && \
    rm libssl1.0.0.deb && \
    apt-get clean && rm -rf /var/lib/apt/lists/*
ENV PATH=$PATH:/usr/lib/rstudio-server/bin

# R packages from the r-studio notebook

RUN python3 -m pip install \
      'jupyter-server-proxy' \
      'jupyter-rsession-proxy' \
      'jupyter-shiny-proxy' && \
    conda install --quiet --yes \   
      'r-base' \
      'r-caret' \
      'r-crayon' \
      'r-devtools' \
      'r-e1071' \
      'r-forecast' \
      'r-hexbin' \
      'r-htmltools' \
      'r-htmlwidgets' \
      'r-irkernel' \
      'r-nycflights13' \
      'r-randomforest' \
      'r-rcurl' \
      'r-rmarkdown' \
      'r-rodbc' \
      'r-rsqlite' \
      'r-shiny' \
      'r-tidyverse' \
      'unixodbc' \
      'r-tidymodels' \
      'r-arrow' \
      'r-aws.s3' \
      'r-catools' \
      'r-devtools' \
      'r-dplyr' \
      'r-ggplot2' \
      'r-hdf5r' \
      'r-jsonlite' \
      'r-odbc' \
      'r-rmarkdown' \
      'r-rstan' \
      'r-sf' \
      'r-shiny' \
      'r-sparklyr' \
      'r-tidyr' \
    && \
    conda clean --all -f -y && \
    fix-permissions.sh $CONDA_DIR && \
    fix-permissions.sh /home/$NB_USER

COPY resources/tools $RESOURCES_PATH/tools

# R-runtimes and Desktop RStudio from remote-desktop-rstudio

RUN /bin/bash $RESOURCES_PATH/tools/r-runtime.sh && \
    /bin/bash $RESOURCES_PATH/tools/r-studio-desktop.sh && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists

# QGIS and geomatics libraries from remote-desktop-geomatics

RUN /bin/bash $RESOURCES_PATH/tools/qgis.sh && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists

# From minimal-notebook-cpu. Partially redundant to r-notebook, kept for reference (conda ignores existing installs)
RUN pip install --quiet --no-cache-dir \
      'jupyterlab-commenting-service' \
      'jupyter-lsp' \
      'kfp-notebook==0.10.1' \
    && \
    conda install --quiet --yes \
    'jupyterlab' \
    && \
    conda install --quiet --yes \
    -c conda-forge \
      'ipywidgets' \
      'ipympl' \
      'jupyter_contrib_nbextensions' \
      'jupyterlab-dash' \
      'jupyterlab-git==0.20.0' \
      'xeus-python' \
      'nodejs' \
      'python-language-server' \
    && \
    conda install --quiet --yes \
      'pyarrow' \
      'r-dplyr' \
      'r-tidyr' \
      'r-jsonlite' \
      'r-ggplot2' \
      'r-rstan' \
    && \
    conda clean --all -f -y && \
    jupyter nbextension enable codefolding/main --sys-prefix && \
    jupyter labextension install --no-build \
      '@ijmbarr/jupyterlab_spellchecker' \
      '@jupyter-widgets/jupyterlab-manager'  \
      '@hadim/jupyter-archive' \
      '@krassowski/jupyterlab-lsp' \
      '@lckr/jupyterlab_variableinspector' \
      '@jupyterlab/commenting-extension' \
      '@jupyterlab/debugger' \
      '@jupyterlab/github' \
      '@jupyterlab/git@0.20.0' \
      '@jupyterlab/toc' \
      'jupyter-matplotlib' \
      'jupyterlab-execute-time' \
      'jupyterlab-plotly' \
      'jupyterlab-theme-solarized-dark' \
      'jupyterlab-spreadsheet' \
      'nbdime-jupyterlab' \
    && \
    jupyter lab build && \
    jupyter lab clean && \
  npm cache clean --force && \
  rm -rf $HOME/.cache/yarn && \
  rm -rf $HOME/.node-gyp && \
  fix-permissions.sh $CONDA_DIR && \
  fix-permissions.sh $HOME

ARG KUBECTL_VERSION=v1.15.10
ARG KUBECTL_URL=https://storage.googleapis.com/kubernetes-release/release/${KUBECTL_VERSION}/bin/linux/amd64/kubectl
ARG KUBECTL_SHA=38a0f73464f1c39ca383fd43196f84bdbe6e553fe3e677b6e7012ef7ad5eaf2b

ARG MC_VERSION=mc.RELEASE.2020-06-20T00-18-43Z
ARG MC_URL=https://dl.min.io/client/mc/release/linux-amd64/archive/${MC_VERSION}
ARG MC_SHA=7198b0634f84c27e656bba527bf80ed048087e99afc7744e0fdd9e0790a726f5

ARG AZCLI_URL=https://aka.ms/InstallAzureCLIDeb 
ARG AZCLI_SHA=c03302f47be07d02afe3edec63080c7806980c51709c016af2f27901d51417b4

# # kubectl, mc, az
RUN curl -LO "${KUBECTL_URL}" \
    && echo "${KUBECTL_SHA} kubectl" | sha256sum -c - \
    && chmod +x ./kubectl \
    && sudo mv ./kubectl /usr/local/bin/kubectl \
  && \
    wget --quiet -O mc "${MC_URL}" \
    && echo "${MC_SHA} mc" | sha256sum -c - \
    && chmod +x mc \
    && mv mc /usr/local/bin/mc \
  && \
    curl -sLO https://aka.ms/InstallAzureCLIDeb \
    && echo "${AZCLI_SHA} InstallAzureCLIDeb" | sha256sum -c - \
    && bash InstallAzureCLIDeb

# Solarized Theme and Cell Execution Time
RUN echo '{ "@jupyterlab/apputils-extension:themes": {"theme": "JupyterLab Solarized Dark"}, "@jupyterlab/notebook-extension:tracker": {"recordTiming": true}}' > /opt/conda/share/jupyter/lab/settings/overrides.json && \
    fix-permissions.sh /home/$NB_USER

RUN pip --no-cache-dir install --quiet minio==5.0.10

### Python testing

RUN conda install -c conda-forge --quiet --yes \
      'pylint' \
      'pytest' \
    && \
    conda clean --all -f -y

ARG SHA256py=a4191fefc0e027fbafcd87134ac89a8b1afef4fd8b9dc35f14d6ee7bdf186348
ARG VS_PYTHON_VERSION=2020.5.86806
RUN rm -rf /tmp/home_nbuser_default/.vscode/extensions/ms-python.python-2020.6.91350 && \
    wget --quiet --no-check-certificate https://github.com/microsoft/vscode-python/releases/download/$VS_PYTHON_VERSION/ms-python-release.vsix && \
    echo "${SHA256py} ms-python-release.vsix" | sha256sum -c - && \
    bsdtar -xf ms-python-release.vsix extension && \
    rm ms-python-release.vsix && \
    mv extension /tmp/home_nbuser_default//.vscode/extensions/ms-python.python-$VS_PYTHON_VERSION


### CUDA Test 1: ml-workspace GPU flavor REF https://github.com/ml-tooling/ml-workspace/blob/master/gpu-flavor/Dockerfile
### NVIDIA CUDA BASE ###
# https://gitlab.com/nvidia/container-images/cuda/blob/ubuntu18.04/10.1/base/Dockerfile
RUN apt-get update && apt-get install -y --no-install-recommends gnupg2 curl ca-certificates && \
    curl -fsSL https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/7fa2af80.pub | apt-key add - && \
    echo "deb https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64 /" > /etc/apt/sources.list.d/cuda.list && \
    echo "deb https://developer.download.nvidia.com/compute/machine-learning/repos/ubuntu1804/x86_64 /" > /etc/apt/sources.list.d/nvidia-ml.list && \
    # Cleanup - cannot use cleanup script here, otherwise too much is removed
    apt-get clean && \ 
    rm -rf $HOME/.cache/* && \
    rm -rf /tmp/* && \
    rm -rf /var/lib/apt/lists/*

ENV CUDA_VERSION 10.1.168
ENV CUDA_PKG_VERSION 10-1=$CUDA_VERSION-1

# For libraries in the cuda-compat-* package: https://docs.nvidia.com/cuda/eula/index.html#attachment-a
RUN apt-get update && apt-get install -y --no-install-recommends \
        cuda-cudart-$CUDA_PKG_VERSION \
        cuda-compat-10-1 && \
    ln -s cuda-10.1 /usr/local/cuda && \
    rm -rf /var/lib/apt/lists/* && \
    # Cleanup - cannot use cleanup script here, otherwise too much is removed
    apt-get clean && \ 
    rm -rf $HOME/.cache/* && \
    rm -rf /tmp/* && \
    rm -rf /var/lib/apt/lists/*

# Required for nvidia-docker v1
RUN echo "/usr/local/nvidia/lib" >> /etc/ld.so.conf.d/nvidia.conf && \
    echo "/usr/local/nvidia/lib64" >> /etc/ld.so.conf.d/nvidia.conf

ENV PATH /usr/local/nvidia/bin:/usr/local/cuda/bin:${PATH}
ENV LD_LIBRARY_PATH /usr/local/nvidia/lib:/usr/local/nvidia/lib64:${LD_LIBRARY_PATH}

# nvidia-container-runtime
# https://github.com/NVIDIA/nvidia-container-runtime#environment-variables-oci-spec
ENV NVIDIA_VISIBLE_DEVICES all
ENV NVIDIA_DRIVER_CAPABILITIES compute,utility
ENV NVIDIA_REQUIRE_CUDA "cuda>=10.1 brand=tesla,driver>=384,driver<385 brand=tesla,driver>=396,driver<397 brand=tesla,driver>=410,driver<411"

### CUDA RUNTIME ###
# https://gitlab.com/nvidia/container-images/cuda/blob/ubuntu18.04/10.1/runtime/Dockerfile

ENV NCCL_VERSION 2.4.2

RUN apt-get update && apt-get install -y --no-install-recommends \
        cuda-libraries-$CUDA_PKG_VERSION \
        cuda-nvtx-$CUDA_PKG_VERSION \
        libnccl2=$NCCL_VERSION-1+cuda10.1 && \
    apt-mark hold libnccl2 && \
    rm -rf /var/lib/apt/lists/* && \
    # Cleanup - cannot use cleanup script here, otherwise too much is removed
    apt-get clean && \ 
    rm -rf $HOME/.cache/* && \
    rm -rf /tmp/* && \
    rm -rf /var/lib/apt/lists/*

### END CUDA RUNTIME ###

### CUDA DEVEL ###
# https://gitlab.com/nvidia/container-images/cuda/blob/ubuntu18.04/10.1/devel/Dockerfile
RUN apt-get update && apt-get install -y --no-install-recommends \
        cuda-libraries-dev-$CUDA_PKG_VERSION \
        cuda-nvml-dev-$CUDA_PKG_VERSION \
        cuda-minimal-build-$CUDA_PKG_VERSION \
        cuda-command-line-tools-$CUDA_PKG_VERSION \
        libnccl-dev=$NCCL_VERSION-1+cuda10.1 && \
    rm -rf /var/lib/apt/lists/* && \ 
    # Cleanup - cannot use cleanup script here, otherwise too much is removed
    apt-get clean && \ 
    rm -rf /root/.cache/* && \
    rm -rf /tmp/* && \
    rm -rf /var/lib/apt/lists/*

ENV LIBRARY_PATH /usr/local/cuda/lib64/stubs

### END CUDA DEVEL ###

### CUDANN7 DEVEL ###
# https://gitlab.com/nvidia/container-images/cuda/blob/ubuntu18.04/10.1/devel/cudnn7/Dockerfile

ENV CUDNN_VERSION 7.6.0.64
LABEL com.nvidia.cudnn.version="${CUDNN_VERSION}"

RUN apt-get update && apt-get install -y --no-install-recommends \
            libcudnn7=$CUDNN_VERSION-1+cuda10.1 \
            libcudnn7-dev=$CUDNN_VERSION-1+cuda10.1 && \
    apt-mark hold libcudnn7 && \
    rm -rf /var/lib/apt/lists/* && \ 
    # Cleanup
    apt-get clean && \ 
    rm -rf /root/.cache/* && \
    rm -rf /tmp/* && \
    rm -rf /var/lib/apt/lists/*

### END CUDANN7 ###

# Link Cupti:
ENV LD_LIBRARY_PATH ${LD_LIBRARY_PATH}:/usr/local/cuda/extras/CUPTI/lib64

# Install TensorRT. Requires that libcudnn7 is installed above.
# https://www.tensorflow.org/install/gpu#ubuntu_1804_cuda_101
RUN apt-get update && apt-get install -y --no-install-recommends libnvinfer6=6.0.1-1+cuda10.1 \
    libnvinfer-dev=6.0.1-1+cuda10.1 \
    libnvinfer-plugin6=6.0.1-1+cuda10.1 && \
    # Cleanup
    clean-layer.sh

## GPU DATA SCIENCE LIBRARIES ###

RUN \
    apt-get update && \
    apt-get install -y libomp-dev libopenblas-base && \
    # Not needed? Install cuda-toolkit (e.g. for pytorch: https://pytorch.org/): https://anaconda.org/anaconda/cudatoolkit
    conda install -y cudatoolkit=10.1 -c pytorch && \
    # Install cupy: https://cupy.chainer.org/
    pip install --no-cache-dir cupy-cuda101 && \
    # Install pycuda: https://pypi.org/project/pycuda
    pip install --no-cache-dir pycuda && \
    # Install gpu utils libs
    pip install --no-cache-dir gpustat py3nvml gputil && \
    # Install scikit-cuda: https://scikit-cuda.readthedocs.io/en/latest/install.html
    pip install --no-cache-dir scikit-cuda && \
    # Install tensorflow gpu - conda uninstall removes too much and conda remove corrupts environment
    # only tensorflow 2.1 supports cuda 10.1
    pip uninstall -y tensorflow && \
    pip install --no-cache-dir tensorflow-gpu==2.1.0 && \
    # Install ONNX GPU Runtime
    pip uninstall -y onnxruntime && \
    pip install --no-cache-dir onnxruntime-gpu==1.1.1 && \
    # Install pytorch gpu
    # https://pytorch.org/get-started/locally/
    conda install -y pytorch torchvision -c pytorch && \
    # Update mxnet to gpu edition
    pip uninstall -y mxnet-mkl && \
    pip install --no-cache-dir mxnet-cu101mkl==1.5.1.post0 && \
    # install jax: https://github.com/google/jax#pip-installation
    pip install --no-cache-dir --upgrade jax https://storage.googleapis.com/jax-releases/cuda101/jaxlib-0.1.37-cp37-none-linux_x86_64.whl && \
    # Install pygpu - Required for theano: http://deeplearning.net/software/libgpuarray/
    conda install -y pygpu && \
    # nvidia python ml lib
    pip install --upgrade --force-reinstall nvidia-ml-py3 && \ 
    # SpeedTorch: https://github.com/Santosh-Gupta/SpeedTorch
    pip install --no-cache-dir SpeedTorch && \ 
    # Cleanup
    # Cleanup python bytecode files - not needed: https://jcrist.github.io/conda-docker-tips.html
    find ${CONDA_DIR} -type f -name '*.pyc' -delete && \
    find ${CONDA_DIR} -type l -name '*.pyc' -delete && \
    clean-layer.sh

# https://www.anaconda.com/getting-started-with-gpu-computing-in-anaconda/

# By default, the majority of GPU memory will be allocated by the first
# execution of a TensorFlow graph. While this behavior can be desirable for
# production pipelines, it is less desirable for interactive use. Set
# TF_FORCE_GPU_ALLOW_GROWTH to change this default behavior as if the user had
ENV TF_FORCE_GPU_ALLOW_GROWTH true

### END DATA SCIENCE LIBRARIES ###

### GPU TOOLS ###

# Install Glances & Netdata GPU Support
RUN \
    apt-get update -y && \
    apt-get install lm-sensors -y && \
    apt-get install netcat -y && \
    apt-get install iproute2 -y && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
    git clone https://github.com/Splo0sh/netdata_nv_plugin --depth 1 /tmp/netdata_nv_plugin && \
    cp /tmp/netdata_nv_plugin/nv.chart.py /usr/libexec/netdata/python.d/ && \
    cp /tmp/netdata_nv_plugin/python_modules/pynvml.py /usr/libexec/netdata/python.d/python_modules/ && \
    cp /tmp/netdata_nv_plugin/nv.conf /etc/netdata/python.d/ && \
    # Cleanup
    clean-layer.sh

## END GPU TOOLS ###

### Fixes a potential obscure permissions glitch

RUN mv $HOME/.config $HOME/.config2 && \
    mv $HOME/.config2 $HOME/.config && \
    fix-permissions.sh $HOME

## Workspace overwrite hotfix

RUN cp -r --no-clobber $HOME /tmp/home_nbuser_default && \
    chown -R $NB_USER /tmp/home_nbuser_default

## Landing page

COPY resources/nginx.conf /etc/nginx/nginx.conf
COPY resources/landing_page $RESOURCES_PATH/landing_page

COPY init.sh /init.sh
RUN chmod +x /init.sh

RUN rm -rf $HOME
RUN chown -R $NB_USER $HOME

USER ${NB_USER}

CMD ["/init.sh"]
