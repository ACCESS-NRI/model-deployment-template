# Creating Pre/Releases For A Model Deployment Repository

This model deployment repository makes use of the [spack](https://spack.readthedocs.io/en/latest/) build and deployment functionality enabled by the [access-nri/build-cd repository](https://github.com/ACCESS-NRI/build-cd).

This allows automated Release and Prerelease `spack` builds of climate models, informed by this repositories `spack.yaml` file and `config` directory.

In a nutshell, the commits on Pull Requests to this model deployment repository generate isolated Prerelease builds of a climate model on HPCs. When these are merged, an official Release build is created.

FIXME: Add in the image of the process

## The `spack.yaml` File

The `spack.yaml` file at the root of the repository is a file understood by `spack`, that essentially constrain the versions and features of dependencies required for a build of a given climate model. It is a set of abstract constraints that is _concretized_ into a single set of dependencies at build time, creating a `spack.lock` file.

It is similar to other package managers abstract/concrete paradigm, like `npm`s `package.json`/`package.lock` files.

More information on the `spack.yaml` file itself can be found on [ACCESS-NRI's devdocs](https://github.com/ACCESS-NRI/dev-docs/wiki/Spack#the-spackyaml-file-spec-file).

## The `config` Directory

The `config` directory at the root of this repository contains a single `versions.json` file, which allows customisation of both the version of [`access-nri/spack`](https://github.com/ACCESS-NRI/spack) that the model will be deployed on, as well as the version of [`access-nri/spack-packages`](https://github.com/ACCESS-NRI/spack-packages) that will source the ACCESS-NRI spack packages.

## The PR Process

1. The first step is to create a branch off the model deployment repositories `main`.
2. [Modifications](#modifications-to-the-spackyaml-file) can be made to the `spack.yaml`/`config/versions.json` files as applicable - update versions of packages, add variants, etc.
3. Once a commit has been made that you want to build, open a PR into `main` in the model deployment repository, and the CI/CD infrastructure will attempt to build.
    * If there are CI errors in the build, they will be reported via GitHubs Pull Request Checks. More information on them is available by clicking the failing test.
    * If the CI checks succeed, a bot will comment on what is being deployed on the open pull request.
    * Finally, GitHubs Environment Deployment dialogue will show on the Pull Request as a yellow 'In Progress' label. Once it shows a green 'Active' (or grey 'Inactive') deployment, you can use the modules specified in the above comment - they should look something like `MODEL/prX-Y` for the `MODEL`s `X`th PR and `Y`th deployment.
4. Further commits can be put onto the Pull Request to create more builds, without removing the earlier ones, allowing side-by-side testing. For example, adding an extra commit will allow use of the module `MODEL/prX-(Y+1)` once it's deployed.
5. When the Pull Request is closed, whether it is merged or not, the Prereleases **will be removed**, but can be recreated at your leisure by installing the `spack.yaml`/`spack.lock` artifact at the specified commit on your own instance, or by creating a new Pull Request.
6. If the Pull Request is merged, it will create an offical ACCESS-NRI released build of the given model, with an associated GitHub Release.

## Modifications To The `spack.yaml` File

FIXME: Write up common modifications to the `spack.yaml`