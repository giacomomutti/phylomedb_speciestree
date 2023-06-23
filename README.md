# PhylomeDB species tree reconstruction pipeline v 0.0.1

## Download data

First, you need to download all phylomedb useful data to run the pipeline. This means the besttrees, aln and info file.

To do this you just need to add the ids you are interested in to `data/ids/phy_ids.txt`

And then run:

`snakemake -s pipeline/download_data.smk -p -c2 -k`

After this completed successfully you can go forward. **NOTE:** Do this step in mn0 or in a mounted folder as you need internet!

You can see phylome ids, seed species and description in this file:  [PhylomeDB.csv](data/info/PhylomeDB.csv) (Downloaded: 20/6/23).

**NOTE:** Some phylomes have weird alignmet or tree file, the pipeline should manage to identify the weird ones, add the ids in `data/info/failed_download.ids` and avoid adding the id in the sptree reconstruction pipeline untile the data issue has been resolved. This is the reason of the `-k --keep-going` option.

## Methods

| method        | multicopy | branch_lengths | branch_support       | root | source                                                                         |
| ------------- | --------- | -------------- | -------------------- | ---- | ------------------------------------------------------------------------------ |
| duptree       | yes       | no             | no                   | yes  | https://doi.org/10.1093/bioinformatics/btn230                                  |
| astral        | no        | CU             | localPP              | no   | https://bmcbioinformatics.biomedcentral.com/articles/10.1186/s12859-018-2129-y |
| W-astral      | no        | CU             | localPP              | no   | https://doi.org/10.1093/molbev/msac215                                         |
| Astral-DISCO  | yes       | CU             | localPP              | no   | https://doi.org/10.1093/sysbio/syab070                                         |
| Astral-PRO    | yes       | CU             | localPP              | no   | https://doi.org/10.1093/bioinformatics/btac620                                 |
| SpeciesRax    | yes       | sub/site       | EQPIC score          | yes  | https://doi.org/10.1093/molbev/msab365                                         |
| Fast-Mul-Rfs  | yes       | ?              | no                   | no   | https://doi.org/10.1093/bioinformatics/btaa444                                 |
| Asteroid      | yes       | distance       | bootstrap from input | no   | https://doi.org/10.1093/bioinformatics/btac832                                 |
| Concatenation | yes/no    | sub/site       | bootstrap,gCF,sCF    | no   |                                                                                |

Possible additions:

* mp-est (requires rooted single copy gene trees in nexus formats)
* NJst, metal, steac, star from phybase package


## Usage

![smk_pipeline](data/dags/sptree_reconstruction.png)

The pipeline takes the data previously downloaded, parse them and apply 8 different summary species tree reconstruction methods: see [methods](data/info/methods.csv)

After all 8 trees are computed you can get a consensus tree with support computed as `% of methods agreeing/gCF/sCF`. [Gene concordance factor and site conconrdance factor](http://www.iqtree.org/doc/Concordance-Factor) are computed with single copy gene trees with minimum occupancy of 10% (by default, it can be set in this [config file](data/configs/pdb_config.yml))

## MareNostrum

Highmem is usually better as sometimes fastmulrfs and asteroid may fail due to memory issues.

```
#!/bin/bash    
#SBATCH --job-name=pdb_sp
#SBATCH --output=logs/pdb_sp.out
#SBATCH --error=logs/pdb_sp.err
#SBATCH --cpus-per-task=48
#SBATCH --time=2:00:00

#SBATCH --qos=debug
#SBATCH --constraint=highmem

module load ANACONDA/2022.10
conda activate snakemake

ulimit -s 2000000
snakemake -s pipelines/sptree_reconstruction.smk --unlock
snakemake -p -s pipelines/sptree_reconstruction.smk --cores 48 --printshellcmds --rerun-incomplete --keep-going
```

## Outputs


## Runtimes

These are the runtimes for some test phylomes:

![smk_runtimes](data/img/runtimes.png)


### TODO

* Check if other reason aln tar or tree file could be weird!
* option to root with species2age
* plot and filter gene trees based on exploratory statistics
* if method fails go ahead and use less than 8 tree
* compare patristic distance and correlation
* discovista similar stuff to visualize gene tree discordances and different species trees discordance
* tree space analysis
* benchmark qfo
* ideally split rules in parsing and runtimes to get the right runtimes
* installation of tools

