# classify_raw_ccs
walkthrough multiple options for classifying pacbio ccs reads

## kraken2
both building the databases and running the classification can use a lot of memory, which is why i'm using large memory nodes

```
#!/bin/bash

#SBATCH --time=20:00:00
##SBATCH --account=epscor-condo
#SBATCH --partition=bigmem
#SBATCH -N 1 # Single Node
#SBATCH -n 2 # Two cores 
#SBATCH --mem-per-cpu=300G
##SBATCH --threads-per-core=2
#SBATCH -J set_up_nt_db_kraken

#----- End of slurm commands ----

#load modules
module load blast/2.7.1+

#point to kraken2 scripts
KRAKEN_DIR=/users/ibishop/git/kraken2

#download library via ftp
$KRAKEN_DIR/kraken2-build --download-library nr --db nr_kraken --use-ftp --protein

#download taxonomy
$KRAKEN_DIR/kraken2-build --download-taxonomy --db nr_kraken

#build database
kraken2-build --build --db nr_kraken --threads 1
```


Now that you've successfully built your databases, use them!
```
#!/bin/bash

#SBATCH --time=00:30:00
##SBATCH --account=epscor-condo
#SBATCH --partition=bigmem
#SBATCH -N 1 # Single Node
#SBATCH -n 2 # Two cores 
#SBATCH --mem-per-cpu=150G
##SBATCH --threads-per-core=2

#----- End of slurm commands ----

#point to database
DBNAME=~/data/ibishop/dbs_kraken/nr_kraken

#point to reads to classify
CCS_READS=/users/ibishop/scratch/ncbi_blasting_ccs/ccs.fa

#load relevant modules 
module load blast/2.7.1+

#classify with nt database
#kraken2 --threads 2 --report nt_classify_ccs_raw_report --use-names --output nt_classify_ccs_raw --db $DBNAME $CCS_READS

#classify with nr database
kraken2 --threads 2 --report nr_classify_ccs_raw_report --use-names --output nr_classify_ccs_raw --db $DBNAME $CCS_READS
```

## tiara

```

```

## DIAMOND blastx

```

```

## Kaiju

```

```

