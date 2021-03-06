# SCORPiOs - Synteny-guided CORrection of Paralogies and Orthologies

[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0) [![Snakemake](https://img.shields.io/badge/snakemake-≥5.5.4-brightgreen.svg)](https://snakemake.bitbucket.io)


 SCORPiOs is a synteny-guided gene tree correction [Snakemake](https://snakemake.readthedocs.io/en/stable/) pipeline. SCORPiOs builds optimized gene trees, consistent with a known WGD event, local synteny context, as well as gene sequence evolution.

 For a complete description of SCORPiOs, see our preprint: https://www.biorxiv.org/content/10.1101/2020.01.30.926915v1.full

## Table of content
  - [Installation](#installation)
    - [Installing conda](#installing-conda)
    - [Installing SCORPiOs](#installing-scorpios)
  - [Usage](#usage)
    - [Running SCORPiOs on example data](#running-scorpios-on-example-data)
      - [Example 1: Simple SCORPiOs run](#example-1-simple-scorpios-run)
      - [Example 2: SCORPiOs in iterative mode](#example-2-scorpios-in-iterative-mode)
    - [Running SCORPiOS on your data](#running-scorpios-on-your-data)
    - [Understanding SCORPiOs outputs](#understanding-scorpios-outputs)
      - [Basic](#basic)
      - [Advanced](#advanced)
  - [Authors](#authors)
  - [License](#license)
  - [References](#references)

## Installation

### Installing conda

The Miniconda3 package management system manages all SCORPiOs dependencies, including python packages and other software.

To install Miniconda3:

- Download Miniconda3 installer for your system [here](https://docs.conda.io/en/latest/miniconda.html)

- Run the installation script: `bash Miniconda3-latest-Linux-x86_64.sh` or `bash Miniconda3-latest-MacOSX-x86_64.sh`, and accept the defaults

- Open a new terminal, run `conda update conda` and press `y` to confirm updates

### Installing SCORPiOs

- Clone the repository and go to SCORPiOs root folder
```
git clone https://github.com/DyogenIBENS/SCORPIOS.git
cd SCORPIOS
```

- Create the main conda environment (solving dependencies may take a while, ~ up to an hour)
```
conda env create -f envs/scorpios.yaml
```

## Usage

### Running SCORPiOs on example data

Before any SCORPiOs run, you should:
 - go to SCORPiOs root folder,
 - activate the conda environment with `conda activate scorpios`.

#### Example 1: Simple SCORPiOs run

Inputs and parameters to execute SCORPiOs have to be specified in a configuration file.
An example configuration file is provided: [config_example.yaml](config_example.yaml).

This configuration file allows to execute SCORPiOs on toy example data located in [data/example/](data/example/), that you can use as reference for input formats. SCORPiOs can either start from an input gene trees forest or build one using TreeBeST. More details on input files and formats can be found in [config_example.yaml](config_example.yaml).

In brief, SCORPiOs input files are:
- A single file with the gene trees forest to correct OR a genes to species mapping file to build a starting forest
- A single file with the corresponding multiple alignments
- Gene coordinates files
- A species tree

The only required snakemake arguments to run SCORPiOs are `--configfile` and the `--use-conda` flag. Optionally, you can specify the number of threads via the `--cores` option. For more advanced snakemake usage, you can look at the [Snakemake documentation](https://snakemake.readthedocs.io/en/stable/).

To run SCORPiOs on example data:

```
snakemake --configfile config_example.yaml --use-conda --cores 4
```

#### Example 2: SCORPiOs in iterative mode

SCORPiOs can run in iterative mode, meaning that SCORPiOs improves gene trees a first time, and then uses the corrected forest as input for a new correction run. Correcting gene trees improves orthologies accuracy, which in turn makes synteny conservation patterns more informative, allowing to better integrate it into the gene tree reconstruction. Usually, a small number of iterations (2-3) suffice to reach convergence.

To run SCORPiOs in iterative mode, you can execute the wrapper bash script `iterate_scorpios.sh`:

```
bash iterate_scorpios.sh --j=example --snake_args="--configfile config_example.yaml"
```

Command-line arguments:

```
Required
--j=JOBNAME, specifies scorpios jobname, should be the same as in config
--snake_args="SNAKEMAKE ARGUMENTS", should at minimum contain --configfile

Optional
--max_iter=MAXITER, maximum number of iterations to run, default=5.
--min_corr=MINCORR, minimum number of corrected sub-trees to continue to the next iteration, default=1.
--starting_iter=ITER, starting iteration, to resume a run at a given iteration, default=1.
```

### Running SCORPiOs on your data
After correct formatting of your input data (see [config_example.yaml](config_example.yaml)), you have to create a new configuration file, using the provided example:

- Copy the example config file `cp config_example.yaml config.yaml`
- Open and edit `config.yaml`

If you want to check your configuration, you can execute a dry-run with `-n`.
```
snakemake --configfile config.yaml --use-conda -n
```

Finally, you can run SCORPiOs as described above.

```
snakemake --configfile config.yaml --use-conda
```

or, assuming the jobname is set to 'myjobname' in the new config:

```
bash iterate_scorpios.sh --j=myjobname --snake_args="--configfile config.yaml"
```

### Understanding SCORPiOs outputs

#### Basic

All outputs of a SCORPiOs run are stored in a folder named SCORPiOs_jobname, with the jobname specified in the config file.

The main output is the **SCORPiOs corrected gene trees forest**. In the corrected forest, SCORPiOs adds correction tags that allow to inspect corrections with external gene tree visualisation software. The commands above generate `SCORPiOs_example/SCORPiOs_corrected_forest_0.nhx` for the simple run and `SCORPiOs_example/SCORPiOs_corrected_forest_2_with_tags.nhx` for the iterative run. When in iterative mode, outputs are suffixed with a digit representing the iteration number. This number is set to 0 in simple mode and starts at 1 in iterative mode.

Some intermediary outputs are also stored in different sub-folders (see below for a detailed description). In addition, SCORPiOs writes statistics on key steps of the workflow to the standard output. Thus, to separate output statistics from snakemake logs, you can run:

```
snakemake --configfile config_example.yaml --use-conda >out 2>err
```

or

```
bash iterate_scorpios.sh --j=example --snake_args="--configfile config_example.yaml" >out 2>err
```

#### Advanced

To go beyond description statistics printed to the standard output, one might want to further investigate results of a SCORPiOs run, for one or several given gene families. This section quickly introduces key SCORPiOs concepts, in order to better understand intermediary outputs.

A gene family in SCORPiOs consists of an outgroup gene and all gene copies in WGD duplicated species. Gene families are first defined in an outgroup/duplicated species orthology table. For each family, SCORPiOs computes a synteny-derived orthology graph, then a constrained tree topology based on synteny, and finally, if necessary, a synteny-aware corrected tree. Through each of these steps, a gene family is identified by the outgroup gene name.

The Orthology table is stored in a single file in the `Families/` sub-folder. Raw synteny-predicted orthologies are stored in a single file in `Synteny/`. Predicted orthology groups based on community detection in synteny graphs are stored in a single file in `Graphs/`, along with a summary of the community detection step. Finally, `Corrections/` stores two files, one detailing trees vs synteny consistency and another with the list of successfully corrected trees.

Several tags such as the name of the corrected WGD, the outgroup species used and SCORPiOs iteration number are added to each output file, in order to precisely identify outputs, even for complex configuration.

Additional files can be saved if specified in the configuration file, see [config_example.yaml](config_example.yaml) for details.

## Authors
* [**Elise Parey**](mailto:elise.parey@bio.ens.psl.eu)
* **Alexandra Louis**
* **Hugues Roest Crollius**
* **Camille Berthelot**

## License

This code may be freely distributed and modified under the terms of the GNU General Public License version 3 (GPL v3)
- [LICENSE GPLv3](LICENSE.txt)

## References

SCORPiOs uses the following tools to build and test gene trees:

- [ProfileNJ](https://github.com/maclandrol/profileNJ): Noutahi et al. (2016) Efficient Gene Tree Correction Guided by Genome Evolution. PLOS ONE, 11, e0159559.

- [PhyML](http://www.atgc-montpellier.fr/phyml/): Guindon et al. (2010) New Algorithms and Methods to Estimate Maximum-Likelihood Phylogenies: Assessing the Performance of PhyML 3.0. Syst Biol, 59, 307–321.

- [TreeBeST](https://github.com/Ensembl/treebest): Vilella et al. (2009) EnsemblCompara GeneTrees: Complete, duplication-aware phylogenetic trees in vertebrates. Genome Res., 19, 327–335.

- [CONSEL](https://github.com/shimo-lab/consel): Shimodaira and Hasegawa (2001) CONSEL: for assessing the confidence of phylogenetic tree selection. Bioinformatics, 17, 1246–1247.
