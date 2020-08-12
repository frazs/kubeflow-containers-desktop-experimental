FROM k8scc01covidacr.azurecr.io/remote-desktop-test:vscode-hotfix-base

USER root

# To-do test/upstream: clean-layer modified to not delete /tmp/home_nbuser_desktop, which exists from the base image
# COPY resources/clean-layer.sh clean-layer.sh

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
# Excludes Elyra: needs troubleshooting
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

# SQLite drivers -- not included here, packages do not match.
# The default /etc/odbcinst.ini file doesn't use full paths,
# and it was causing R to fail to load the drivers.
# RUN apt-get update && apt-get install -y \
#     libsqlite3-dev \
#     libsqliteodbc \
#     && apt-get clean && rm -rf /var/lib/apt/lists/* && \
#     sed -i 's~^\(Setup\|Driver\)=\([^/]*\)$~\1=/usr/lib/x86_64-linux-gnu/odbc/\2~' /etc/odbcinst.ini && \
#     sed -i 's~libsqliteodbc.so~libsqlite3odbc.so~' /etc/odbcinst.ini 

ARG KUBECTL_VERSION=v1.15.10
ARG KUBECTL_URL=https://storage.googleapis.com/kubernetes-release/release/${KUBECTL_VERSION}/bin/linux/amd64/kubectl
ARG KUBECTL_SHA=38a0f73464f1c39ca383fd43196f84bdbe6e553fe3e677b6e7012ef7ad5eaf2b

ARG MC_VERSION=mc.RELEASE.2020-06-20T00-18-43Z
ARG MC_URL=https://dl.min.io/client/mc/release/linux-amd64/archive/${MC_VERSION}
ARG MC_SHA=7198b0634f84c27e656bba527bf80ed048087e99afc7744e0fdd9e0790a726f5

ARG AZCLI_URL=https://aka.ms/InstallAzureCLIDeb 
ARG AZCLI_SHA=c03302f47be07d02afe3edec63080c7806980c51709c016af2f27901d51417b4

# ARG PACHCTL_VERSION=1.10.5
# ARG PACHCTL_URL=https://github.com/pachyderm/pachyderm/releases/download/v${PACHCTL_VERSION}/pachctl_${PACHCTL_VERSION}_amd64.deb 
# ARG PACHCTL_SHA=8bc58206e87c3c4304a074c1479e08bdf173743dd6869d0ad4e792830a0df5c5

# # kubectl, mc, az, pachctl
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
#   && \
#     curl -o /tmp/pachctl.deb -L "${PACHCTL_URL}" \
#     && echo "${PACHCTL_SHA} /tmp/pachctl.deb" | sha256sum -c - \
#     && sudo dpkg -i /tmp/pachctl.deb

# Solarized Theme and Cell Execution Time
RUN echo '{ "@jupyterlab/apputils-extension:themes": {"theme": "JupyterLab Solarized Dark"}, "@jupyterlab/notebook-extension:tracker": {"recordTiming": true}}' > /opt/conda/share/jupyter/lab/settings/overrides.json && \
    fix-permissions.sh /home/$NB_USER

# # Go
# ENV GOROOT=/usr/local/go
# ENV PATH=$PATH:$GOROOT/bin
# ENV GOPATH=/opt/go/

# ARG GO_VERSION=1.14.4
# ARG GO_URL=https://dl.google.com/go/go${GO_VERSION}.linux-amd64.tar.gz
# ARG GO_SHA=aed845e4185a0b2a3c3d5e1d0a35491702c55889192bb9c30e67a3de6849c067

# RUN wget "${GO_URL}" -O /tmp/go.tar.gz && \
#     echo "${GO_SHA} /tmp/go.tar.gz" | sha256sum -c - && \
#     tar -xvf /tmp/go.tar.gz -C /usr/local && \
#     printf "\nexport GOROOT=$GOROOT"             | tee -a /etc/profile && \
#     printf "\nexport PATH=\$GOROOT/bin:\$PATH\n" | tee -a /etc/profile && \
#     printf "\nexport GOPATH=$GOPATH\n"           | tee -a /etc/profile /home/$NB_USER/.bashrc && \
#     printf "\nexport PATH=$GOPATH/bin:\$PATH\n"  | tee -a /etc/profile /home/$NB_USER/.bashrc
# RUN mkdir -p $GOPATH && \
#     env GO111MODULE=off go get -d -u github.com/gopherdata/gophernotes && \
#     cd "$(go env GOPATH)"/src/github.com/gopherdata/gophernotes && \
#     env GO111MODULE=on go install && \
#     mkdir -p /opt/conda/share/jupyter/kernels/gophernotes && \
#     cp kernel/* /opt/conda/share/jupyter/kernels/gophernotes && \
#     cd /opt/conda/share/jupyter/kernels/gophernotes && \
#     chmod +w ./kernel.json && \
#     sed "s|gophernotes|$(go env GOPATH)/bin/gophernotes|" < kernel.json.in > kernel.json && \
#     fix-permissions.sh /home/$NB_USER && \
#     fix-permissions.sh $GOPATH

# # From base-notebook, sans pytorch, tensorflow, dremio
# RUN pip --no-cache-dir install --quiet \
#       'kfp==1.0.0' \
#       'kfp-server-api==1.0.0' \
#       'kfp-tekton==0.2.0' \
#       'kubeflow-fairing==1.0.1' \
#       'kubeflow-metadata==0.3.1' \
#       'minio==5.0.10' \
#       'git+https://github.com/zachomedia/s3fs@8aa929f78666ff9e323cde7d9be9262db5a17985'

# # kfp-azure-databricks needs to be run after kfp
# RUN pip --no-cache-dir install --quiet \
#       'fire==0.3.1' \
#       'git+https://github.com/kubeflow/pipelines@1d86111d8f152d3ed7506ea59cee1bfbc28abbf9#egg=kfp-azure-databricks&subdirectory=samples/contrib/azure-samples/kfp-azure-databricks'

RUN pip --no-cache-dir install --quiet minio==5.0.10

### Fixes a potential obscure permissions glitch

RUN mv $HOME/.config $HOME/.config2 && \
    mv $HOME/.config2 $HOME/.config && \
    fix-permissions.sh $HOME

## Workspace overwrite hotfix

RUN cp -r $HOME /tmp/home_nbuser_default && \
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
