# indico-rpms-fedora

Repository with scripts and configs for Indico packaging


## How does the packaging work in CI

The CI jobs are executed in two containers based on Fedora39 and Debian12.
Those containers include software exclusively from the standard Fedora and Debian repositories.
It should be possible to use the container images for the local development.

The "missing" software needed for Indico is build in CI from the sources via packaging into DEB or RPMs.

All the packaging and installation runs sequential in the scrips `rpm_alllocal.sh` and  `deb_alllocal.sh`.
During the installation no packages from the standard repositories are downloaded, no upgrades are done, etc.
The end result of the execution of those scripts is a set of DEB and RPM packages, which togather with the packages 
from the standard repositories form a fully functional Indico installation.

## How does the packaging works for individual package
The packaging of individual package into RPM or DEB binary is done with
`srpmsbuild.sh` and `debbuild.sh` packages correspondinly. 

The `srpmsbuild.sh` script takes two arguments - normalized package name `p` and version `v`.
The `debbuild.sh` script takes also a third argument, the original package name `P`.


Internally, the scripts look into RPM/p/v or DEB/p/v directories, 
guess the source location for the package, download it, repackage, patch and build.
Therefore, one needs an internet connection to execute them successfully.

The `srpmsbuild.sh` script also executes the RPM/p/v/do.sh script, if such a script is present.
Typically such a script is handy for creation of patches.

The `debbuild.sh` script also executes the DEB/p/v/get.sh script, if such a script is present.
Typically such a script is handy for downloads and repackaging of sources.


## Security
The `srpmsbuild.sh` script  checks the MD5 sum of the downloaded sources. The allowed checksums are stored in 
RPM/md5sums.txt If the md5sum in not found in that file, the build will not start.

The same measures for the `debbuild.sh` script is WIP.

## Interdependencies of packages

All RPM packages have implemented dependencies.
The repository contains two packages which are quite technical.
- `indico-devel` -- a package w/o sources, but includes almost all dependencies needed to buils/install Indico.
This package is a dependency of the main package `python-indico`.

- `indico-mpp` -- a package that include configuration of Indico specific for MPP.
Depends on  `python-indico`.

The dependencies of the DEB packages is WIP.


## YUM/APT repositories.

With the supplies SRC RPM and DEB packages it is possible to create APT or YUM 
repository either locally or on any well-known platform.

The RPM packages are present in the COPR in the `https://copr.fedorainfracloud.org/coprs/averbyts/I330` YUM repository.
Practically this means that on Fedora39 the commands 

```
dnf -y install dnf-plu*
dnf -y copr enable averbyts/I330
yum -y install python3-indico 
yum -y install python3-indico-mpp-configuration 

```

will install a fully functional Indico instance.

The COPR repository can be re-created using the script RPM/submit.sh. The `copr-cli` utility and COPR account is required.

