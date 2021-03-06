# `conda`
This is a compilation of a few terminal commands for managing different python environments using `conda`, with a brief introduction also pointing to the alternatives of `conda` environments. As a disclaimer I will use `conda` and `anaconda` as synonyms, the former is the abbreviated command you use inside your terminal.

## Introduction Code Reproducibility

Every code has a couple of dependencies that limit its reproducibility/portability

### Hard-coded dependencies
These are the obvious dependencies which are easily avoided by adhering to good coding practices.

- file paths
- random numbers or randomized code outcome

### Software dependencies
 Often your code will not run on another machine even if code and underlying data source have not been changed because of variations in the installation of the programming languages that you have been using. A typical data science project can depend on a number of installations on your machine. To name a few of them

- `R` *
- `R packages` *
- `python` *
- `python packages` *
- `pip`
- `git`
- `Java`
- `a C++ compiler`
- `pandoc`
- `latex`
- `phantom js`

The most important being `R` and `python` and the underlying packages which usually shoulder the workload of your code.

### System/Hardware dependencies
Then there is you machine in itself which has a specific get up:

- OS (32bit, 64bit)
- environment variable
- Hardware (number of cores, RAM, GPU)


### Programming language-based solutions
There are a number of strategies to reproducibly preserve either your `python` or your `R` environment. For R there is `packrat` and `checkpoint` and for `python` there is the `pip freeze` command, which preserves all packages and version in a `requirements.txt` file.

However these solutions are focused on snapshotting the programming language environment and do not consider any of the other tools and system configurations that might be necessary to run your code. 

### Environment-based solutions (`anaconda`)
`conda` environmnents use a bit of a more holistic approach and strive to encapsulate apps and programming languages including the packages. They provide a strategy for handling `R` and `python` simultaneously and they have a set of packages for which they offer their own support for. Meaning you can download the packages from a `conda` server and they are garanteed to be compatible with each other as long as they are being installed at the same time. However the `conda` environments are not garanteed to be platform independent. The list of supported packages depends on the OS. However anaconda will make an effort to replicate the environment as closely as possible. 

- [list of all `python` packages managed by `anaconda`](https://docs.anaconda.com/anaconda/packages/pkg-docs)  
- [list of all `R` packages managed by `anaconda`](https://docs.anaconda.com/anaconda/packages/r-language-pkg-docs)


### Container-based solutions
The state of the art solution is a `docker` container, which containerizes everything (tools, drivers, environment variables, etc), this eases pushing code into procuction  as explained [here](https://www.dataquest.io/blog/docker-data-science/). There are even solutions for systems with specialized hardware configurations such as GPUs, as explained [here](https://indico.io/data-science-deployments-docker/)

# Managing `anaconda` environments

## `anaconda` navigator GUI

after installing the anaconda distribution you can run the navigator app, which allows you to create environments and manage
the installed packages. As always with these tools some commands will only work in the command line.

# Command Line 

condensated version of official [documentation](https://conda.io/docs/user-guide/tasks/manage-environments.html#removing-an-environment)

[conda cheat sheet](https://conda.io/docs/_downloads/conda-cheatsheet.pdf)

## Check installation
```
conda --version
```

## Create an environment
that contains `python` and `R`, we are specifiying r-base because conda will install the microsoft R distribution by default.
```
conda create --name myenv python=3.6 r-base=3.4.3
```


## Activate environment
will add your environment to your command line
```
conda activate myenv
```

on older versions of conda
will add your environment to your command line
```
source activate myenv
```

## Deactivate environment
will remove your environment from your command line

```
conda deactivate
```
## Remove an environment
```
conda remove -name myenv --all
```

## Install packages and apps

### Install MacOS SDK
`anaconda` not only contains packages but also apps and other programs. It also contains a set of compilers. For licensing reasons the MacOS SDK cannot be included and needs to be installed seperately as explained [here](https://conda.io/docs/user-guide/tasks/build-packages/compiler-tools.html). MacOS SDK can be downloaded [here](https://github.com/phracker/MacOSX-SDKs) I suggest to download the version that matches the version number of your installed macOS.

Add the following code to your `conda_build_config.yaml`
```
CONDA_BUILD_SYSROOT:
  - /opt/MacOSX10.9.sdk        # [osx]
```


### Channels
An important issue for `anaconda` are the channels where we can find tha packages that we want to install in our `anaconda` environments. Each environment contains a list of channels which will be searched from top to bottom for a package that we would like to install. `anaconda` will always prioritize the channel over the version number thus might not always install the latest versions. Each environment starts with the channel `defaults` which are the official channels

##### Add a channel to the bottom of the list
You want to make sure that the defaults channels rank high thus I recommend appending all new channels.
```
conda config --append channels conda-forge bioconductor
```

##### Add a channel to the top of the list
here you want to add your own channel containing your anaconda builds. 
```
conda config --prepend channels anaconda_username
```

### Install packages without build/compilation

*make sure the environment that you want to install to is active*

*we can install apps such as Rstudio, pip, etc (see anaconda launcher with the same synthax)*

*do not use `R; install.packages()`, packages installed that way cannot be tracked*


#### Install from listed channel
```
conda install package_name
```

#### Install from pip
```
conda install pip
pip install package_name
pip install git+repos_url
```

### Install packages with build/compilation
This will be mostly required for `R` packages that cannot be found on `anaconda` ressources which unfortunately is true for most of the packages. Managing your `R` packages with `anaconda` takes a long time to set up if you already start with a large set of package requirements and can be efficient if you gradually build it up as you go. However you will not be cross platform compatible. You will have to more or less go through the same iterative steps on each platform you want to be compatible with. The reason is that `R` packages containing `C++` or `Fortran` code need different compilers depending on the operating system that you use. Some of these compilers are not referenced directly but they reference to another `R` package which is at the base of the dependency trail. There is the option of converting you package builds compiled on one platform to another which will work for the majority of the packages but not for all.

#### Workflow outline for creating an `R` environment

1. create an anaconda account where you will upload your packages to
2. enable automatic uploading after package building
3. create an environment inlcuding python, R, and r-essentials.
4. add personal R channel to the top of you channels list
5. add bioconductor and conda-forge to the end of your channels list
6. identify a package that you want to add to your environment which is not available from an anaconda source
7. download a skeleton from CRAN
8. build/compile the package
    - anaconda will create test environment in which the package will be built based on the requirements saved in the skeleton. For `R` packages it will only detect dependencies in the `DESCRIPTION` file.  
    - anaconda will look for the required packages in the channels and try to install whatever it finds. It will list all packages and sources it will install from. If it cannot find the necessary packages it will fail.  
    - anaconda might raise a dependency conflict remove the package you suspect to be the oldest and skeleton/build install again.  
9. Troubleshoot if compiling/building fails
    - Try to skim the error message for the incompatible package, however the error message is not always meaningful  
    - `R` Packages from conda-forge and bioconductor are likely to be outdated get them from CRAN as skeleton instead  
    - `R` packages have the `R` version that they were compiled for attached to the version number. Check if that matches the `r-base` version of your environment  
  - Packages that come from you own repository but have initially been built for another platform and contain `C++` or `Fortran` code must likely be rebuilt from skeleton for your present platform.
10. If building/compilation was successfull the package will be uploaded to our anaconda repository from where it can be installed
11. repeat those steps until all packages have been successfully installed
12. if you can run some test code, download the source code with the highest number of dependencies and run the inlcuded test.
13. export an yml file and try to recreate the environment from it.
14. Consider to repeat this process for the other platorms as well if you want your code to be completely portable

**This process is very tedious however it will always work once you identify the incompatibilities**
 
 #### List of R packages that contain `C++` or `fortran` and are not part of r-essentials
 - fastmatch
 - ff
 - ffbase
 - bit
 - spam
 - maps
 - dotcall64
 - fields
 - spam
 - mda
 - party
 - cubist
 - kknn
 - gamlss
 - rlang (part of r-essentials but better to install the newest version)
 
 
#### Create an anaconda account and login, activate automated upload
create the account on [anaconda.org](https://anaconda.org)

```
anaconda anaconda login
conda config --set anaconda_upload yes
# anaconda logout
```

#### Download skeleton form CRAN/github

#### github release tags
anaconda will automatically checkout out the master branch with the latest release tag of packages from github. Often repository owners do not assign release tags or only if a major version increment has been made. Therefore it is best to fork the repository to a personal public github account and then apply any missing release tags.

#### github dependencies
`R` packages from github do not have to pass any quality checks as they need to for being uploaded to CRAN. Therefore it can happen that not all dependencies from the `NAMESPACE` file are also listed in the `DESCRIPTION` file. In such a case the testing environment for creating a conda build will not be sufficient.

```
conda skeleton cran rlang
conda skeleton cran https://github.com/erblast/ggplot2.git
```

#### Build packages from skeleton

we have to ad a `r-` prefix to the package name and put the package name in *lowercase*
```
conda build r-rlang
```

#### Check package installations

```
conda list -name myenv pandas
```

#### Exporting an environment to .yml
activate the environment that you want to export. It will save a `.yml` file into the current working directory
```
conda env export -f myenv.yml
```

#### Build environment based on .yml file
```
conda env create -f environment.yml
```

#### Convert packages to other formats
This did only work for me when trying to convert from OS-64 to linux-64, Then I could not use any of the converted packages that contained `C++` or `Fortran` code even if it seemed to work at the beginning.

*Change to your conda-bld directory in the shell*

```
for f in *.bz2; do conda convert -f --platform linux-64 -o .. $f; done
```

