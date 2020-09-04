# kubeflow-containers-desktop-experimental
This repository defines experimental extensions of the [Remote Desktop images](https://github.com/StatCan/kubeflow-containers-desktop). \
The intent is for a fast and loose testing ground, or to provide guidelines for individual specialized customizations. \
These images are neither officially supported nor intended for general or long-term production use. \
They may be behind on updates, as well as have decreased features, stability, and/or security.

## remote-desktop-jupyterlab
Combines all flavours of [Remote Desktop](https://github.com/StatCan/kubeflow-containers-desktop) with JupyterLab as well as additions implemented in
[r-studio-cpu](https://github.com/StatCan/kubeflow-containers/blob/master/r-studio/Dockerfile),
[minimal-notebook-cpu](https://github.com/StatCan/kubeflow-containers/blob/master/minimal-notebook/cpu/Dockerfile),
and [the ml-workspace GPU flavour](https://github.com/ml-tooling/ml-workspace/blob/master/gpu-flavor/Dockerfile).

Excludes Elyra, kale, Go, Julia, and pachyderm integrations.

Too large for GitHub Actions CI; must be built/pushed manually.

This image will most likely be deprecated or repurposed after:
- A GPU flavor of Remote Desktop is developed
- Storage sharing is implemented between the individual components
