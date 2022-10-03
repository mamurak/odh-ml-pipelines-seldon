# Table of contents
1. [Getting started](#getting-started)
    * [Prerequisites](#Prerequisites)
    * [Deploying the environment](#deploying-the-environment)
2. [Folder structure](#folder-structure)
4. [References](#references)

# Getting started

## Prerequisites

* OpenShift Container Platform version 4.10 or above.
* OpenShift Pipelines operator.
* OpenShift GitOps operator.
* Seldon Core operator.
* S3 compliant object store, e.g. OpenShift Data Foundation.

## Deploying the environment

### Install Open Data Hub

* Create a new project `odh-applications`.
* Install the Open Data Hub operator from the Operator Hub.
* Select Open Data Hub operator in Installed Operators within project `odh-applications`.
* Deploy `manifests/odh.yaml`.
* Deploy `manifests/pipelines.yaml`.
* After all components have been deployed, scale down the `opendatahub-operator` pod to 0 in the `openshift-operators` project. This is required so we can freely configure the ODH components.
* Adapt and deploy `manifests/odh-jupyterhub-admins.yaml`, defining all users that should receive ODH admin permissions.
* Verify the deployment by opening the `odh-dashboard`route URL. You should see the ODH dashboard. If your user is included in the ODH admin group, you should see the `Settings` tab in the dashboard.

### Set up image builds

* Deploy the imagestream manifests located in `manifests/odh/images/` (files with `-is.yaml` suffix).
* Deploy the build config manifests located in `manifests/odh/images/` (files with `-bc.yaml` suffix).
* Trigger builds of the new build configs.

### Set up custom notebook

* As an ODH admin user, open the `Settings` tab in the ODH dashboard.
* Select `Custom notebook`.
* Add new notebook with image URL `custom-notebook:latest` and appropriate metadata.
* Verify custom notebook integration in the JupyterHub provisioning page. You should be able to provision an instance of the custom notebook that you have defined in the previous step.

# How-To

* Changing the available deployment sizes of Jupyter pods.
    * Adapt and deploy `manifests/odh/odh-jupyterhub-sizes.yaml` in the `odh-applications` project.
* Adding packages to custom notebook image with pinned versions.
    * Within a custom notebook instance, install the package through `pip install {your-package}`.
    * Note the installed version of the package.
    * Add a new entry in `container-images/custom-notebook/requirements.txt` with `{your-package}=={installed-version}`.
    * Trigger a new image build.
    * Once the build is finished, provision a new notebook instance using the custom notebook image. The new package is now available.

# Troubleshooting

* In case you're using custom certificates in your environment, you might not be able to access the JupyterHub entry page (HTTP error 599). To work around this issue, do the following:
    * In the `jupyterhub-cfg` configmap, set the `jupyterhub_config.py` parameter to `c.OpenShiftOAuthenticator.validate_cert = False`.
    * Restart the JupyterHub pods.
* If you don't have cluster admin privileges and you're denied access to the ML Pipelines UI route, you may have to update the oauth proxy arguments in the `ml-pipeline-ui` deployment. Refer to `manifests/odh/ml-pipeline-ui-deployment.yaml` for details.

# Folder structure

* `notebooks`: scripts and sample procedures for ODH component integration.
* `manifests`: OpenShift deployment artifacts.
* `container-images`: dependencies of container builds.

# References

* [OpenShift Container Platform](https://docs.openshift.com/container-platform/4.10/welcome/index.html)
* [Open Data Hub](https://opendatahub.io/) ([Github](https://github.com/opendatahub-io/odh-manifests))
* [JupyterHub](https://jupyterhub.readthedocs.io/en/latest/)
