# ASKCOS
Software package for the prediction of feasible synthetic routes towards a desired compound and associated tasks related to synthesis planning. Originally developed under the DARPA Make-It program and now being developed under the [MLPDS Consortium](http://mlpds.mit.edu).

# Overview
Starting with version 2020.07, the ASKCOS project is now comprised of five GitHub repositories.

| Repository | Description |
| :--------- | :---------- |
| [![askcos-base](https://img.shields.io/badge/-askcos--base-blue?style=flat-square)](https://github.com/ASKCOS/askcos-base) | provides compiled dependencies, currently RDKit |
| [![askcos-core](https://img.shields.io/badge/-askcos--core-blue?style=flat-square)](https://github.com/ASKCOS/askcos-core) | provides the base Python package, including various models and utilities |
| [![askcos-data](https://img.shields.io/badge/-askcos--data-blue?style=flat-square)](https://github.com/ASKCOS/askcos-data) | provides data and models needed by `askcos-core`, uses `git-lfs` |
| [![askcos-site](https://img.shields.io/badge/-askcos--site-blue?style=flat-square)](https://github.com/ASKCOS/askcos-site) | provides web interface and additional tools |
| [![askcos-deploy](https://img.shields.io/badge/-askcos--deploy-blue?style=flat-square)](https://github.com/ASKCOS/askcos-deploy) | provides scripts for deploying `askcos-site` using Docker Compose or Kubernetes |

# Public Site
A publicly accessible version of ASKCOS can be explored at [askcos.mit.edu](https://askcos.mit.edu).
Please note that this is a shared resource, so please be considerate of other users when submitting tasks.
In addition, please do not submit any proprietary data to the public site as it should not be considered secure.
If you have any questions or comments, please contact askcos_support@mit.edu.

# Releases
- [2021.01](/2021-01/)
    - [API v2 Reference](/2021-01/api2)
- [2020.07](/2020-07/)
    - [API v2 Reference](/2020-07/api2)
- [0.4.1](/0-4-1/)

# License
All of the ASKCOS code is licensed under MPL 2.0. However, the data and models are licensed under CC BY-NC-SA (for non-commercial use only).
