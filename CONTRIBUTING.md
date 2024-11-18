# Creating Pre/Releases For A Model Deployment Repository

This model deployment repository makes use of the [spack](https://spack.readthedocs.io/en/latest/) build and deployment functionality enabled by the [access-nri/build-cd repository](https://github.com/ACCESS-NRI/build-cd).

This allows automated Release and Prerelease `spack` builds of climate models, informed by this repositories `spack.yaml` file and `config` directory.

In a nutshell, the commits on Pull Requests to this model deployment repository generate isolated Prerelease builds of a climate model on HPCs. When these are merged, an official Release build is created.

FIXME: Add in the image of the process

## The `spack.yaml` File

The `spack.yaml` file at the root of the repository is a file understood by `spack`. It constrains the versions and features (variants) of dependencies required to build a given climate model. It is a set of abstract constraints that is _concretized_ into a single set of dependencies at build time, creating a `spack.lock` file.

It is similar to `npm`s `package.json`/`package.lock` files.

More information on the `spack.yaml` file itself can be found on [ACCESS-NRI's devdocs](https://github.com/ACCESS-NRI/dev-docs/wiki/Spack#the-spackyaml-file-spec-file).

## The `config` Directory

The `config` directory at the root of this repository contains a single `versions.json` file, which allows customisation of both the version of [`access-nri/spack`](https://github.com/ACCESS-NRI/spack) that the model will be deployed on, as well as the version of [`access-nri/spack-packages`](https://github.com/ACCESS-NRI/spack-packages) that will source the ACCESS-NRI spack packages recipes.

## The PR Process

1. The first step is to create a branch off the model deployment repositories `main`.
2. [Modifications](#modifications-to-the-spackyaml-file) can be made to the `spack.yaml`/`config/versions.json` files as applicable - update versions of packages, add variants, etc.
3. Once a commit has been made that you want to build, open a PR into `main` in the model deployment repository, and the CI/CD infrastructure will attempt to build.
    * If there are CI errors in the build, they will be reported via GitHubs Pull Request Checks. More information on them is available by clicking the failing test.
    * If the CI checks succeed, a bot will comment on what is being deployed on the open pull request.
    * Finally, GitHubs Environment Deployment dialogue will show on the Pull Request as a yellow 'In Progress' label. Once it shows a green 'Active' (or grey 'Inactive') deployment, you can use the modules specified in the above comment - they should look something like `MODEL/prX-Y` for the `MODEL`s `X`th PR and `Y`th deployment.
4. Further commits can be put onto the Pull Request to create more builds, without removing the earlier ones, allowing side-by-side testing. For example, adding an extra commit will allow use of the module `MODEL/prX-(Y+1)` once it's deployed.
5. When the Pull Request is closed, whether it is merged or not, the Prereleases **will be removed**, but can be recreated at your leisure by installing the `spack.yaml`/`spack.lock` artifact at the specified commit on your own instance, or by creating a new Pull Request.
6. If the Pull Request is merged, it will create an official ACCESS-NRI released build of the given model, with an associated GitHub Release.

## Modifications To The `spack.yaml` File

Below are some real-world examples of how to modify the `spack.yaml` file.

### Example Modification Workflow of `ACCESS-OM2`s `MOM5` Package

In this example, we are using the `ACCESS-OM2` deployment repositories `spack.yaml` from [this commit](https://github.com/ACCESS-NRI/ACCESS-OM2/blob/47bc7bf979c1dfa12a24272cb739117abc50d7ca/spack.yaml). We are also modifying one of ACCESS-OM2's model components, namely [ACCESS-NRI/MOM5](https://github.com/ACCESS-NRI/MOM5). This model component is available in our [`ACCESS-NRI/spack-packages`](https://github.com/ACCESS-NRI/spack-packages) as the package `mom5`.

1. We clone and make our modifications to the model component `ACCESS-NRI/MOM5`, which we tag with [`dev_2024.08.14`](https://github.com/ACCESS-NRI/MOM5/releases/tag/dev_2024.08.14).
2. We then create a branch in the model deployment repository `ACCESS-NRI/ACCESS-OM2` - such as `update_mom5_dev_build`.
3. We verify that the versions of `spack` and `spack-packages` in [`config/versions.json`](https://github.com/ACCESS-NRI/ACCESS-OM2/blob/47bc7bf979c1dfa12a24272cb739117abc50d7ca/config/versions.json) are as we want them.
4. We then make our modifications to the `spack.yaml` file:
   * Firstly, we bump the version of the [overall deployment](https://github.com/ACCESS-NRI/ACCESS-OM2/blob/47bc7bf979c1dfa12a24272cb739117abc50d7ca/spack.yaml#L8). This version encompasses all changes done in a `spack.yaml`, in the form `CALVER_YEAR.CALVER_MONTH.MINOR`:
      * Since this is a minor change, we will just bump the minor version. Hence, we update to `@git.2024.03.1`.
      * When we bump the version, we also need to update the [associated module projection](https://github.com/ACCESS-NRI/ACCESS-OM2/blob/47bc7bf979c1dfa12a24272cb739117abc50d7ca/spack.yaml#L51) to `{name}/2024.03.1` as well.
   * Secondly, we bump the version of `mom5` that we updated earlier, to the `dev_2024.08.14` tag set in the model component repository:
      * We update the version of the [`mom5` package](https://github.com/ACCESS-NRI/ACCESS-OM2/blob/47bc7bf979c1dfa12a24272cb739117abc50d7ca/spack.yaml#L15) to `@git.dev_2024.08.14`.
      * We also update the [associated module projection](https://github.com/ACCESS-NRI/ACCESS-OM2/blob/47bc7bf979c1dfa12a24272cb739117abc50d7ca/spack.yaml#L53) to `{name}/VERSION-{hash:7}` - in this case, `{name}/dev_2024.08.14-{hash:7}`. We use the `{hash:7}` so it doesn't conflict with other versions.
5. Now that we have completed our modifications, we commit them and open a Pull Request in `ACCESS-NRI/ACCESS-OM2` into `main`, even if it's just a dev build.
6. The CI/CD kicks off, we get a comment saying our build is deploying, and once we see that the deployment is 'Active', we go onto the HPC and `module use/load` the module specified in the comment. Excellent!
