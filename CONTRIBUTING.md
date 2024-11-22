# Creating Pre/Releases For A Model Deployment Repository

This model deployment repository makes use of the [spack](https://spack.readthedocs.io/en/latest/) build and deployment functionality enabled by the [access-nri/build-cd repository](https://github.com/ACCESS-NRI/build-cd).

This allows automated Release and Prerelease `spack` builds of climate models, informed by this repository's `spack.yaml` file and `config` directory.

In a nutshell, the commits on Pull Requests to this model deployment repository generate isolated Prerelease builds of a climate model on HPCs. When these are merged, an official Release build is created.

FIXME: Add in the image of the process

## The `spack.yaml` File

The `spack.yaml` file at the root of the repository is a file understood by `spack`. It constrains the versions and features (variants) of dependencies required to build a given climate model. It is a set of abstract constraints that are _concretized_ into a single set of dependencies at build time, creating a `spack.lock` file.

It is similar to `npm`'s `package.json`/`package.lock` files.

More information on the `spack.yaml` file itself can be found on [ACCESS-NRI's devdocs](https://github.com/ACCESS-NRI/dev-docs/wiki/Spack#the-spackyaml-file-spec-file).

## The `config` Directory

The `config` directory at the root of this repository contains a single `versions.json` file, which allows customisation of both the version of [`access-nri/spack`](https://github.com/ACCESS-NRI/spack) that the model will be deployed on, as well as the version of [`access-nri/spack-packages`](https://github.com/ACCESS-NRI/spack-packages) that will source the ACCESS-NRI spack packages recipes.

## Changing the Model via Pull Request (PR)

1. The first step is to create a branch from the model deployment repository's `main` branch.
2. [Modifications](#modifications-to-the-spackyaml-file) can be made to the `spack.yaml`/`config/versions.json` files as applicable - update versions of packages, add variants, etc.
3. Once a commit has been made that you want to build, open a PR into `main` in the model deployment repository, and the CI/CD infrastructure will attempt to build.
    * If there are CI errors in the build, they will be reported via GitHubs Pull Request Checks. More information on them is available by clicking the failing test.
    * If the CI checks succeed, a bot will comment on what is being deployed on the open pull request.
    * Finally, GitHubs Environment Deployment dialogue will show on the Pull Request as a yellow 'In Progress' label. Once it shows a green 'Active' (or grey 'Inactive') deployment, you can use the modules specified in the above comment - they should look something like `MODEL/prX-Y` for the `MODEL`'s `X`th PR and `Y`th deployment.
4. Further commits can be pushed onto the Pull Request branch to create more builds, without removing the earlier ones, allowing side-by-side testing. For example, adding an extra commit will allow use of the module `MODEL/prX-(Y+1)` once it's deployed.
5. When the Pull Request is closed, whether it is merged or not, the Prereleases **will be removed**, but can be recreated by installing the `spack.yaml`/`spack.lock` artifact at the specified commit on your own instance, or by creating a new Pull Request.
6. If the Pull Request is merged, it will create an official ACCESS-NRI released build of the given model, with an associated GitHub Release.

## Modifications To The `spack.yaml` File

Below are some real-world examples of how to modify the `spack.yaml` file.

### Example Modification Workflow of `ACCESS-OM2`'s `MOM5` Package

In this example, we are using the `ACCESS-OM2` deployment repository's `spack.yaml` from [this commit](https://github.com/ACCESS-NRI/ACCESS-OM2/blob/47bc7bf979c1dfa12a24272cb739117abc50d7ca/spack.yaml). We are also modifying one of ACCESS-OM2's model components, namely [ACCESS-NRI/MOM5](https://github.com/ACCESS-NRI/MOM5). This model component is available in our [`ACCESS-NRI/spack-packages`](https://github.com/ACCESS-NRI/spack-packages) as the package `mom5`.

1. Clone and make modifications to the model component `ACCESS-NRI/MOM5`, which is tagged with [`dev_2024.08.14`](https://github.com/ACCESS-NRI/MOM5/releases/tag/dev_2024.08.14).
2. Create a branch in the model deployment repository `ACCESS-NRI/ACCESS-OM2` - such as `update_mom5_dev_build`.
3. Make modifications to the `spack.yaml` file:
   * Firstly, bump the version of the [overall deployment](https://github.com/ACCESS-NRI/ACCESS-OM2/blob/47bc7bf979c1dfa12a24272cb739117abc50d7ca/spack.yaml#L8). This version encompasses all changes done in a `spack.yaml`, in the form `CALVER_YEAR.CALVER_MONTH.MINOR`:
      * Since this is a minor change, just bump the minor version. Hence, it is updated to `@git.2024.03.1`.
      * When that version is bumped, the [associated module projection](https://github.com/ACCESS-NRI/ACCESS-OM2/blob/47bc7bf979c1dfa12a24272cb739117abc50d7ca/spack.yaml#L51) also needs to be updated to `{name}/2024.03.1` as well.
   * Secondly, bump the version of `mom5` that was updated earlier, to the `dev_2024.08.14` tag set in the model component repository:
      * Update the version of the [`mom5` package](https://github.com/ACCESS-NRI/ACCESS-OM2/blob/47bc7bf979c1dfa12a24272cb739117abc50d7ca/spack.yaml#L15) to `@git.dev_2024.08.14`.
      * Also update the [associated module projection](https://github.com/ACCESS-NRI/ACCESS-OM2/blob/47bc7bf979c1dfa12a24272cb739117abc50d7ca/spack.yaml#L53) to `{name}/VERSION-{hash:7}` - in this case, `{name}/dev_2024.08.14-{hash:7}`. The `{hash:7}` is used so the module doesn't conflict with other versions.
4. Now that the modifications are complete, commit the changes and open a Pull Request in `ACCESS-NRI/ACCESS-OM2` into `main`, even if it's just a dev build.
5. The CI/CD kicks off, and [a comment](https://github.com/ACCESS-NRI/ACCESS-OM2/pull/86#issuecomment-2477781588) is added by `github-actions[bot]` saying that the build is deploying, and once the deployment is 'Active', the module specified in the comment is available on the HPC and is `module use/load`able. Excellent!

**NOTE:** If required, when making modifications, verify that the versions of `spack` and `spack-packages` in [`config/versions.json`](https://github.com/ACCESS-NRI/ACCESS-OM2/blob/47bc7bf979c1dfa12a24272cb739117abc50d7ca/config/versions.json) are as required by use-case.
