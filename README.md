# conda
this repository contains conda environments yml files for different platforms that contain python and R including all R packages to run oetteR 0.2.8. . They will be updated as they keep being used. The goal is to have a set of compatible packages for each platform that can be used from within jupyter notebooks. A copy of the yml file of the environment used for a specific analysis or project should always be kept within the project folder.

**Note that the packages depend on the custom anaconda builds that I made for many R packages and have uploaded to my [public ancaonda account](https://anaconda.org/erblast/dashboard)**

## Recreate a conda environment
```
conda env create -f py36r343_linux64.yml
```
