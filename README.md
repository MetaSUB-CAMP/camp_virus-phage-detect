# Virus-Phage Detection

[![Documentation Status](https://img.shields.io/badge/docs-unknown-yellow.svg)]() ![Version](https://img.shields.io/badge/version-0.2.0-brightgreen)

<!-- [![Documentation Status](https://img.shields.io/readthedocs/camp_virus-phage-detect)](https://camp-documentation.readthedocs.io/en/latest/virus-phage-detect.html) -->

## Overview

This module is designed to function as both a standalone viral investigation pipeline as well as a component of the larger CAMP metagenomics analysis pipeline. As such, it is both self-contained (ex. instructions included for the setup of a versioned environment, etc.), and seamlessly compatible with other CAMP modules (ex. ingests and spawns standardized input/output config files, etc.). 

The processed sequencing reads are assembled with MetaSPAdes, and viral contigs are subsequently identified using the output assembly graph and ViralVerify. Contigs containing putative viral genetic material are also identified using VIBRANT, VirSorter, and VirFinder. The aggregated lists of contigs from the three inference algorithms is dereplicated using VirClust and merged with the ViralVerify list, and the overall quality of the putative viruses is assessed using CheckV. 

## Installation

1. Clone repo from [Github](<https://github.com/MetaSUB-CAMP/camp_virus-phage-detect>).

3. Set up the conda environment (contains Snakemake, Click, and other essentials) using `configs/conda/virus-phage-detect.yaml`. 

4. Update the relevant parameters (if applicable- for example, location of external non-conda tools) in `test_data/parameters.yaml`.

5. Make sure the installed pipeline works correctly. 
```Bash
# Create and activate conda environment 
cd camp_virus-phage-detect
conda env create -f configs/conda/virus-phage-detect.yaml
conda activate camp_virus-phage-detect
# Run tests on the included sample dataset
python /path/to/camp_virus-phage-detect/workflow/virus-phage-detect.py test
```

## Using the Module

**Input**: `/path/to/samples.csv` provided by the user.

**Output**: 1) An output config file summarizing 2) the module's outputs. 

- `/path/to/work/dir/virus-phage-detect/final_reports/{sample}_quality_summary.tsv .csv`


### Module Structure
```
└── workflow
    ├── Snakefile
    ├── virus-phage-detect.py
    ├── utils.py
    ├── __init__.py
    └── ext/
        └── scripts/
```
- `workflow/virus-phage-detect.py`: Click-based CLI that wraps the `snakemake` and other commands for clean management of parameters, resources, and environment variables.
- `workflow/Snakefile`: The `snakemake` pipeline. 
- `workflow/utils.py`: Sample ingestion and work directory setup functions, and other utility functions used in the pipeline and the CLI.
- `ext/`: External programs, scripts, and small auxiliary files that are not conda-compatible but used in the workflow.

### Running the Workflow

1. Make your own `samples.csv` based on the template in `configs/samples.csv`. Sample test data can be found in `test_data/`. 
    - `samples.csv` requires either absolute paths or paths relative to the directory that the module is being run in

2. Update the relevant parameters in `configs/parameters.yaml`.

3. Update the computational resources available to the pipeline in `configs/resources.yaml`. 

#### Command Line Deployment

To run CAMP on the command line, use the following, where `/path/to/work/dir` is replaced with the absolute path of your chosen working directory, and `/path/to/samples.csv` is replaced with your copy of `samples.csv`. 
    - The default number of cores available to Snakemake is 1 which is enough for test data, but should probably be adjusted to 10+ for a real dataset.
    - Relative or absolute paths to the Snakefile and/or the working directory (if you're running elsewhere) are accepted!
    - The parameters and resource config YAMLs can also be customized.
```Bash
python /path/to/camp_virus-phage-detect/workflow/mag_qc.py \
    (-c number_of_cores_allocated) \
    (-p /path/to/parameters.yaml) \
    (-r /path/to/resources.yaml) \
    -d /path/to/work/dir \
    -s /path/to/samples.csv
```

If for some reason the module keeps failing, CAMP can print a script called `commands.sh` containing all of the remaining commands that can be run manually. 
```Bash
python /path/to/virus-phage-detect/workflow/virus-phage-detect.py --dry_run \
    -d /path/to/work/dir \
    -s /path/to/samples.csv
```

#### Slurm Cluster Deployment

To run CAMP on a job submission cluster (for now, only Slurm is supported), use the following.
    - `--slurm` is an optional flag that submits all rules in the Snakemake pipeline as `sbatch` jobs. 
    - In Slurm mode, the `-c` flag refers to the maximum number of `sbatch` jobs submitted in parallel, **not** the pool of cores available to run the jobs. Each job will request the number of cores specified by threads in `configs/resources/slurm.yaml`.
```Bash
sbatch -J jobname -o jobname.log << "EOF"
#!/bin/bash
python /path/to/camp_virus-phage-detect/workflow/mag_qc.py --slurm \
    (-c max_number_of_parallel_jobs_submitted) \
    (-p /path/to/parameters.yaml) \
    (-r /path/to/resources.yaml) \
    -d /path/to/work/dir \
    -s /path/to/samples.csv
EOF
```


## Test Dataset 

The test dataset was simulated using [ART](https://www.niehs.nih.gov/research/resources/software/biostatistics/art/index.cfm) with the following parameters:
- read length (-l): 150 
- mean size of fragment (-m): 200 
- standard deviation of fragment (-s): 10.

The following viral species and their corresponding European Nucleotide Archive ([ENA](https://www.ebi.ac.uk/ena/browser/home)) IDs were used as referenced and simulated at coverage (-f):10.
- Corona virus  MN996532
- Dengue virus  KF952702
- Influenza virus   AB262394
- Lambda bacteriophage  AB008394
- M13 bacteriophage AY389148
- Norovirus AB019423
- T4 bacteriophage  JX183125

Additionally, two bacteria were spiked in with coverage (-f):11. Names and NCBI acession IDs below:
- Escherichi acoli GCF_008761535.2_ASM876153v3
- Streptococcus pneumoniae GCA_002076835.1_ASM207683

## Credits
 
- This package was created with [Cookiecutter](https://github.com/cookiecutter/cookiecutter>) as a simplified version of the [project template](https://github.com/audreyr/cookiecutter-pypackage>).
- Free software: MIT License

