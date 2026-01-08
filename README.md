# model-deployment-template

A template repository for the deployment of `spack`-based models.

> [!NOTE]
> Feel free to replace this README with information on the model once the TODOs have been ticked off.

## Common README.md Elements

### Badges for spack, spack-packages, spack-config used

If you want to add badges for the current version of [`spack`](https://github.com/ACCESS-NRI/spack) and [`spack-packages`](https://github.com/ACCESS-NRI/spack-packages) used for the latest model deployment, and the latest version of [`spack-config`](https://github/ACCESS-NRI/spack-config) used across all models, you can add the following urls to your README.

> [!NOTE]
> The first two URLS (`spack` and `spack-packages`) must have the repository name rather than `__MODEL__` (eg, `ACCESS-OM3`), so it can pull data from that repository. The last one (`spack-config`) pulls from a central `build-cd` repository and doesn't need to be edited.

```txt
![Dynamic JSON Badge](https://img.shields.io/badge/dynamic/json?url=https%3A%2F%2Fgithub.com%2FACCESS-NRI%2F__MODEL__%2Fraw%2Fmain%2Fconfig%2Fversions.json&query=%24.spack-packages&label=ACCESS-NRI%2Fspack-packages)

![Dynamic JSON Badge](https://img.shields.io/badge/dynamic/json?url=https%3A%2F%2Fgithub.com%2FACCESS-NRI%2F__MODEL__%2Fraw%2Fmain%2Fconfig%2Fversions.json&query=%24.spack&label=ACCESS-NRI%2Fspack)

![Dynamic JSON Badge](https://img.shields.io/badge/dynamic/json?url=https%3A%2F%2Fgithub.com%2FACCESS-NRI%2Fbuild-cd%2Fraw%2FHEAD%2Fconfig%2Fsettings.json&query=%24.deployment.Gadi.Release.%5B'0.22'%5D.spack-config&label=ACCESS-NRI%2Fspack-config%20(Gadi))
```

If done correctly, they will look something like this (`ACCESS-OM3` example):

![Dynamic JSON Badge](https://img.shields.io/badge/dynamic/json?url=https%3A%2F%2Fgithub.com%2FACCESS-NRI%2FACCESS-OM3%2Fraw%2Fmain%2Fconfig%2Fversions.json&query=%24.spack-packages&label=ACCESS-NRI%2Fspack-packages)

![Dynamic JSON Badge](https://img.shields.io/badge/dynamic/json?url=https%3A%2F%2Fgithub.com%2FACCESS-NRI%2FACCESS-OM3%2Fraw%2Fmain%2Fconfig%2Fversions.json&query=%24.spack&label=ACCESS-NRI%2Fspack)

![Dynamic JSON Badge](https://img.shields.io/badge/dynamic/json?url=https%3A%2F%2Fgithub.com%2FACCESS-NRI%2Fbuild-cd%2Fraw%2FHEAD%2Fconfig%2Fsettings.json&query=%24.deployment.Gadi.Release.%5B'0.22'%5D.spack-config&label=ACCESS-NRI%2Fspack-config%20(Gadi))

## Things TODO to get your model deployed

### Settings

#### Repository Settings

Branch protections should be set up on `main` and the special `backport/*.*` branches, which are used for backporting of fixes to major releases (the `YEAR.MONTH` portion of the `YEAR.MONTH.MINOR` version) of models.

#### Repository Secrets/Variables

There are a few secrets and variables that must be set at the repository level.

##### Repository Secrets

* `GH_COMMIT_CHECK_TOKEN`: GitHub Token that allows workflows to run based on workflow-authored commits (in  the case where a user uses `!bump` commands in PRs that bumps the version of the model)
* `TRACKING_SERVICES_POST_TOKEN`: A token used to post build information about the packages in `config/packages.json` to the release provenance database as part of tracking services, used for Releases.
* `CONFIGS_REPO_TOKEN`: A token used to open PRs to linked configs repositories as part of an `!update-configs` command. Requires `contents:write` and `pull-requests:write`.

##### Repository Variables

* `NAME`: which corresponds to the model name - which is usually the repository name
* `RELEASE_DEPLOYMENT_TARGETS`: Space-separated list of deployment targets when doing release deployments. These are often the names of [keys under the `deployment` key of `build-cd`s `config/settings.json`](https://github.com/ACCESS-NRI/build-cd/blob/09cdf100eefc58f06900e8e9145e77b4caf5a39d/config/settings.json#L3), such as `Gadi` or `Setonix`. As noted [below](#environment-secretsvariables), it is the same as the GitHub Environment name. For example: `Gadi Setonix`
* `PRERELEASE_DEPLOYMENT_TARGETS`: Space-separated list of deployment targets when doing prerelease deployments, similar to the above. For example: `Gadi Setonix` - note the lack of a `Prerelease` specifier!
* `TRACKING_SERVICES_POST_URL`: A url to the API of the release provenance database as part of tracking services, used for Releases.

#### Environment Secrets/Variables

GitHub Environments are sets of variables and secrets that are used specifically to deploy software, and hence have more security requirements for their use.

Currently, we have two Environments per deployment target - one for `Release` and one for `Prerelease`. Our current list of deployment targets and Environments can be found in this [deployment configuration file in `build-cd`](https://github.com/ACCESS-NRI/build-cd/blob/main/config/deployment-environment.json).

In order to deploy to a given deployment target:

* Environments with the name of the deployment target and the type must be created _in this repository_ and have the associated secrets/variables set ([see below](#environment-secrets))
* There must be a `Prerelease` Environment associated with the `Release` Environment. For example, if we are deploying to `SUPERCOMPUTER`, we require Environments with the names `SUPERCOMPUTER Release`, `SUPERCOMPUTER Prerelease`.

When setting the environment up, remember to require sign off by a member of ACCESS-NRI when deploying as a `Release`.

Regarding the secrets and variables that must be created:

##### Environment Secrets

* `HOST`: The deployment location SSH Host
* `HOST_DATA`: The deployment location SSH Host for data transfer (may be the same as `HOST`)
* `SSH_KEY`: A SSH Key that allows access to the above `HOST`/`HOST_DATA`
* `USER`: A Username to login to the above `HOST`/`HOST_DATA`

##### Environment Variables

* `DEPLOYED_MODULES_DIR`: Directory that will contain the modules created during the installation of the model. This can be virtual modules created by a [`.modulerc` file](https://github.com/ACCESS-NRI/build-cd/tree/main/tools/modules) in the directory.
* `DEPLOYMENT_TARGET`: Name of the deployment target. It is exported to the deployment target and used for variations in `spack.yaml` build processes - seen most prominently in mutually-exclusive 'when' clauses like `spack.definitions[].when = env['DEPLOYMENT_TARGET'] == 'gadi'`. Also used for logging purposes.
* `SPACK_INSTALLS_ROOT_LOCATION`: Path to the directory that contains all versions of a deployment of `spack`. For example, if `/some/apps/spack` is the `SPACK_INSTALLS_ROOT_LOCATION`, that directory will contain directories like `0.20`, `0.21`, `0.22`, which in turn contain an install of `spack`, `spack-packages` and `spack-config`
* `SPACK_YAML_LOCATION`: Path to a directory that will contain the `spack.yaml` from this repository during deployment
* (Optional) `SPACK_INSTALL_ADDITIONAL_ARGS`: Additional flags outside of `--fresh --fail-fast` to add to the `spack install` command. For advanced users who need to tailor the installation options in their repository.

### File Modifications

#### In `.github/workflows`

* Reminder that these workflows use `vars.NAME` (as well as inherit the above environment secrets) and hence these must be set.
* If the name of the root SBD for the model (in [`spack-packages`](https://github.com/ACCESS-NRI/spack-packages/tree/main/packages)) is different from the model name (for example, `ACCESS-ESM1.5`s root SBD is `access-esm1p5`), you must uncomment and set the `jobs.*.with.root-sbd` line to the appropriate SBD name, for all `.github/workflows/*.yml` files.
* Check `inputs.config-versions-schema-version` is an appropriate version of the [`config/versions.json` schema](https://github.com/ACCESS-NRI/schema/tree/main/au.org.access-nri/model/deployment/config/versions).
* Check `inputs.config-packages-schema-version` is an appropriate version of the [`config/packages.json` schema](https://github.com/ACCESS-NRI/schema/tree/main/au.org.access-nri/model/deployment/config/packages).
* Check `inputs.spack-manifest-schema-version` is an appropriate version of the [ACCESS-NRI-style `spack.yaml` schema](https://github.com/ACCESS-NRI/schema/tree/main/au.org.access-nri/model/spack/environment/deployment).

#### In `config/versions.json`

* `.spack` must be given a version. For example, it will clone the associated `releases/vVERSION` branch of `ACCESS-NRI/spack` if you give it `VERSION`.
* `.access-spack-packages` should also have a CalVer-compliant tag as the version. See the [associated repo](https://github.com/ACCESS-NRI/access-spack-packages/tags) for a list of available tags.
* Optionally, a list of `.custom-scopes` can be specified from `spack-config`s `custom/cd/` - this is usually for restricted builds.

#### In `config/packages.json`

* `.provenance`: If components of a model are to be kept track of in the release provenance database, their package names must be added to this list. They will also be injected into the `spack.modules.tcl.default` section of the manifest.
* `.injection`: If packages need to be injected automatically in the manifest, their package names must be added to this list.

#### In `config/auto-configs-pr.json`

This file is split into multiple _profiles_. A _profile_ can be thought of as a configs repository, multiple config branches to open PRs into, and what checks to run for each of those config branches. Users specify a particular _profile_ through `!update-configs profile=PROFILE` (eg. `!update-configs profile=qa-only`). If no profile is specified (eg. `!update-configs`) the required `default` profile is used.

An example `config/auto-configs-pr.json` file looks like this:

```json
{
  "$schema": "https://raw.githubusercontent.com/ACCESS-NRI/schema/main/au.org.access-nri/model/deployment/config/auto-configs-pr/1-0-0.json",
  "profiles": {

    "default": {
      "configs_repo": "ACCESS-NRI/access-test-configs",
      "configs": {
        "dev-01deg_jra55_iaf": {
          "checks": {
            "repro": true
          }
        }
      }
    },

    "qa-only": {
      "configs_repo": "ACCESS-NRI/access-test-configs",
      "configs": {
        "dev-01deg_jra55_iaf": {
          "checks": {
            "repro": false
          }
        },
        "dev-01deg_jra55_ryf": {
          "checks": {
            "repro": false
          }
        }
      }
    }
  }
}
```

This means that `!update-configs` invoked on a Prerelease PR for the HEAD prerelease build (for example, `access-test/pr100-2`), will automatically create one PR in `ACCESS-NRI/access-test-configs`, in a feature branch off the `dev-01deg_jra55_iaf` branch, with all changes required to use the prerelease module. Furthermore, it will run `!test repro` on that PR.

Similarly, `!update-configs profile=qa-only` will open two PRs in `ACCESS-NRI/access-test-configs`, in feature branches off the `dev-01deg_jra55_iaf` and `dev-01deg_jra55_ryf` branches respectively. Repro checks will not be run, but QA checks will run as normal.

If you intend to use the `!update-configs` command, the `default` profile must at least be specified.

#### In `.github/CODEOWNERS`

* By default, @CodeGat will be pinged for review for infrastructure changes.
* Maintainers should set CODEOWNERs of the `spack.yaml` manifest to ensure all affected teams are consulted when changes are proposed to that file.

#### In `spack.yaml`

There are a few TODOs for the `spack.yaml`:

* Only do this step if there are variations in compiler/etc across deployment targets:
  * `spack.definitions`: Use the `ROOT_PACKAGE` to define the root SBD. The `ROOT_SPEC` simply combines the `ROOT_PACKAGE` with the other, mutually-exclusive `compiler_target` definition.
  * `spack.specs`: Set the only element in the spec list to `$ROOT_SPEC` - it will be filled in at install time.

Otherwise:

* `spack.definitions`: Set both the `_name` and `_version` reserved variants, which give the deployment it's name and version.
* `spack.specs`: Set the root SBD as the only element of `spack.specs`.
* `spack.packages.*`: In this section, you can specify the versions and variants of dependencies and compilers. Note that the first element of the `spack.packages.*.require` must be only a version. Variants and other configuration can be done on subsequent lines.
* `spack.packages.all`: Can set configuration for all packages. For example, the compiler used, or the target architecture.
