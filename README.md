# classify_raw_ccs
walkthrough multiple options for classifying pacbio ccs reads

## kraken2
Install kraken2
```
git clone https://github.com/DerrickWood/kraken2.git
cd kraken2
./install_kraken2.sh
```

Download and build database, requires lots of memory
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


Now that you've successfully built your databases, use them! Requires good amount of memory
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

Set up new conda environment and install tiara and dependencies
```
module load anaconda/2020.02
conda create --name tiara python=3.8
source activate tiara
conda install numpy pytorch numba joblib
conda install -c conda-forge tqdm biopython skorch
```

Run tiara classification
```
#!/bin/bash

#SBATCH --time=1:00:00
#SBATCH --mem=48G #default is 1 core with 2.8GB of memory
#SBATCH -n 12
#SBATCH --account=epscor-condo
#SBATCH -J tiara_classify

module load anaconda/2020.02
source activate tiara

CONTIGS=~/scratch/ipa_haplotigs_diamond/final_haplotigs.fa

tiara -i $CONTIGS -o out.txt -v --tf all -t 12
```


## DIAMOND blastx

```

```


## Kaiju
Install kaiju
```
git clone https://github.com/bioinformatics-centre/kaiju.git
cd kaiju/src
make
```

Set up database using large memory node
```
#!/bin/bash

#SBATCH --time=20:00:00
##SBATCH --account=epscor-condo
#SBATCH --partition=bigmem
#SBATCH -N 1 # Single Node
#SBATCH -n 2 # Two cores 
#SBATCH --mem-per-cpu=300G
##SBATCH --threads-per-core=2
#SBATCH -J set_up_nr_db_kaiju

#----- End of slurm commands ----
KAIJU_DIR=/users/ibishop/scratch/kaiju/kaiju/bin

$KAIJU_DIR/kaiju-makedb -s nr_euk
```

Run Kaiju classification
```
#!/bin/bash

#SBATCH --time=1:00:00
#SBATCH --mem=144G #default is 1 core with 2.8GB of memory
#SBATCH -n 12
#SBATCH --account=epscor-condo

#SBATCH -J kaiju_nr_classify

#----- End of slurm commands ----

KAIJU_DIR=~/scratch/kaiju/kaiju
CCS_READS=~/data/ccs.fastq.gz

kaiju -t $KAIJU_DIR/nodes.dmp -f $KAIJU_DIR/kaiju-nreukdb/nr_euk/kaiju_db_nr_euk.fmi -i $CCS_READS -o raw_ccs.out -z 12
```

