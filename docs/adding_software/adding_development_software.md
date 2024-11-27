# Adding software to `dev.eessi.io`

!!! warning "`dev.eessi.io` is still in active development and focused on MultiXscale"

    The `dev.eessi.io` repository and functionality is still in its early stages.
    The repository itself and build + deploy procedure for it are functional, but may change
    often for the time being. 
    
    Our focus is currently on including and supporting developers and applications in the 
    [MultiXscale CoE](https://multixscale.eu).

## What is `dev.eessi.io`?

`dev.eessi.io` is the [development repository of EESSI](../repositories/dev.eessi.io.md).

## Adding software

Using `dev.eessi.io` is similar to using EESSI's production repository `software.eessi.io`.
Software builds are triggered by a [bot](https://www.eessi.io/docs/bot/) listening to pull 
requests in [GitHub repositories](https://github.com/search?q=org%3AEESSI+dev.eessi.io&type=repositories). 
These builds require custom easyconfig and easystack files, which should be in specific directories.

To see this in practice, refer to the [dev.eessi.io-example GitHub repository](https://github.com/EESSI/dev.eessi.io-example).
 In this GitHub repository you will find templates for some software installations with the appropriate directory structure, that is:

```
dev.eessi.io-example
├── easyconfigs
└── easystacks
```

### Quick steps to build for `dev.eessi.io`

- Obtain commit ID from GitHub or GitLab repository with source to build.
- Fork the project's `dev.eessi.io` repository on GitHub, or checkout to a new branch if you can do so.
- If needed, prepare an easyconfig template using `--software-commit` and add it to `easyconfigs/`
- Add an easystack file in `easystacks/` that with the easyconfig file above, add the
commit ID to `software-commit` under `options`.
- Open a PR from the fork or branch to the main branch of the application's `dev.eessi.io` GitHub repository.
- Instruct the bot to start a build by adding a comment with `bot: build`.
- Confirm the build worked correctly. If so, you can deploy the software by adding the label `bot:build` to the pull request.
- Once the staging PR is approved, the development build will become available on `dev.eessi.io` in a few minutes!

### Installation details

#### easyconfig files and `--software-commit`
The approach to build and install software is similar to that of `software.eessi.io`. 
It requires one or more easyconfig files. Easyconfig files used for building for `dev.eessi.io`
do not need to be a part of an [EasyBuild release](https://github.com/easybuilders/easybuild-easyconfigs), unlike builds for 
`software.eessi.io`. In this case, the development easyconfigs can be located under `easyconfigs/` in the `dev.eessi.io` repository being used.

To allow for development builds, we leverage the `--software-commit` functionality (requires [EasyBuild](https://easybuild.io/) v4.9.3 or higher). This lets us build a given application from
a specific commit in repository. This can also be done from a fork, by changing the `github_account` field in the easyconfig file. 
We've created a template for `ESPResSo` based on the standard eaasyconfig of the most recent version. The relevant fields are:

``` python
easyblock = 'CMakeMake'

name = 'ESPResSo'
version = 'devel'
versionsuffix = '-%(software_commit)s'

homepage = 'https://espressomd.org/wordpress'
description = """A software package for performing and analyzing scientific Molecular Dynamics simulations."""

github_account = 'espressomd'
source_urls = ['https://github.com/%(github_account)s/%(name)s/archive/']

sources = ['%(software_commit)s.tar.gz']
```

!!! warning "`--software-commit` disables `--robot`"

    Using `--software-commit` disables the use of `--robot`, so make sure that you explicitly include
    new dependencies that might need to be installed. Otherwise, the easyconfig files won't be found.

You can also make additional changes to the easyconfig file, for example, if the new functionality requires new build or
runtime dependencies, patches, configuration options, etc. It's a good idea to try installing from a specific commit locally first,
to at least see if everything is parsed correctly and confirm that the right sources are being downloaded.


While the process to build for `dev.eessi.io` is similar to the one for the [production repository](../repositories/software.eessi.io.md), there 
are a few additional details to keep in mind.

#### Software version

Installations to the EESSI production repository refer to specific versions of applications. However, development builds can't follow the same 
approach as they are most often not pegged to a release. Because of this, it is possible to include a descriptive "version" label to the `version` parameter
in the easyconfig file for a given (set of) installations. 

Note that some applications are built with custom easyblocks, which may
use the `version` parameter to determine how the installation is meant to work (for example, recent versions need to copy files from to a new directory). Make sure
that you account for this, otherwise you may install software differently than intended. If you encounter issues, you can open an issue in our
[support portal](https://gitlab.com/eessi/support#eessi-support-portal).

#### Installing dependencies

Installations in `dev.eessi.io` are done _on top_ of `software.eessi.io`. That means if your development build depends on some application that is
already installed in `software.eessi.io`, then that will simply be used. However, if you need to add a new dependency, then this must included as 
part of the build. That means including an easyconfig file for it, and adding it to the right easystack file.

#### Using commit IDs or tags for `--software-commit`

Installing with `--software-commit` requires that you include either a commit ID or a tag. The installation procedure will use this to
obtain the sources for the build. Because tags can be changed to point to a different commit ID, we recommend you avoid using them and sticking to
the commit ID itself. You can then include this in the `versionsuffix` on your easyconfig file, to generate a unique (though "ugly") module name.

#### Patch files

If your specific development build requires patch files, you should add these to the `easyconfigs/` directory. If the necessary patch is part of an
EasyBuild release, then this may not be necessary, as these will be directly taken from EasyBuild. If it is a new patch that is not on an EasyBuild
release, then include it in the `easyconfigs/` directory.

#### Checksums

EasyBuild's easyconfig files typically contain [checksums](https://docs.easybuild.io/writing-easyconfig-files/?h=checksums#common_easyconfig_param_sources_checksums)
 as their use is highly recommended. By default, EasyBuild will compute the checksums of sources and patch files it needs for
  a given installation, and compare them with the values in the easyconfig file. Because builds for `dev.eessi.io` change much
   more often, hard coded checksums become a problem, as they'd need to be updated with every new build. For this reason, we
    recommend not including checksums in your development easyconfig files (unless you need to, for a specific reason).

#### Easystack files 
After an easyconfig file has been created and added to the `easyconfigs` subdirectory, an [easystack file](https://docs.easybuild.io/easystack-files) that picks it up
needs to be in place so that a build can be triggered.

!!! note "Naming convention for easystack files"

    The easystack files must follow a naming convention and be named something
    like: `software-eb-X.Y.Z-dev.yml`, where X.Y.Z correspond to the EasyBuild
    version used to install the software. Following our example for 
    `ESPREsSo`, it would look like: 
    
    ```yaml
    easyconfigs:
      - ESPResSo-devel-foss-2023a-software-commit.eb:
          options:
            software-commit: 2ba17de6096933275abec0550981d9122e4e5f28 # release 4.2.2
    ```

`ESPResSo-devel-foss-2023a-software-commit.eb` would be the name of the easyconfig file added in our example step above. 
Note the option passing the `software-commit` for the development version that should be built.
For the sake of this example, the chosen commit actually corresponds to the [4.2.2 release of ESPResSo](https://github.com/espressomd/espresso/releases/tag/4.2.2).

### Triggering builds

We use the [EESSI build-test-deploy bot](../bot.md) to handle software builds.
All one needs to do is open a PR with the changes adding the easyconfig and easystack 
files and commenting `bot: build`. This can only be done by previously authorized users. 
The current build cluster for `dev.eessi.io` builds only for the `zen2` CPU microarchitecture, but this is likely to change.

Once a build is complete and the `bot:deploy` label is added, a staging PR can be merged to deploy the
application to the `dev.eessi.io` cvmfs repository. On a system with `dev.eessi.io` mounted, then all
that is left is to `module use /cvmfs/dev.eessi.io/versions/2023.06/modules/all` and try out the software!

There is currently no initialisation script or module for `dev.eessi.io`, but this feature is coming soon.