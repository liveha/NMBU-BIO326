# **BINNING**

Until now, we have: 

1. **Assembled** our gut microbiome metadata from a ONT sequencing run (CANU:https://canu.readthedocs.io/en/latest/)
2. **Polished** the assembled contigs with short reads from a Illumina sequencing run (Racon:https://github.com/isovic/racon)
3. **Evaluated** the quality before and after the polishing step (BUSCO:https://busco.ezlab.org) 

Next up is **BINNING**, that is sorting the contigs based on their composition and coverage ("_fingerprint_"). Contigs that have the same "fingerprint" most likely originate from the same organism, and will end up in the same bin. 
We ofter refer to these bins as Metagenome-Assembled Genomes, or MAGs. 

You will have several binners to choose betweeen, when selecting a software. We will use a binner that we know work well for gut microbiomes: **MetaBAT** (https://bitbucket.org/berkeleylab/metabat/src/master/)


![image](https://user-images.githubusercontent.com/65181082/115441281-f647a900-a210-11eb-80fe-069cb874d79b.png)

First, you should make a new directory for you binning. Make your way `$SCRATCH` using `cd` and: 

```
mkdir MetaGenomeBinningBio326
```

Before you start your binning, you will have to make a file with coverage information that **MetaBAT** will use. This is done by: 
1. **Map** you raw reads to the contigs to calculate the coverage 
2. Generate a **depth file** using a script from metaBAT: jgi_summarize_bam_contig_depths 

_You have 2 sets of raw files (ONT and Illumina), which would you use?_ 
To consider: The contigs are polished using Illumina data. Will the ONT raw reads ("full" of errors) match back to the polished contigs? 

_Can we use the mapping file we generated (using minimap2) in the polishing step?_ 
To consider: At that time, we mapped the Illumina raw reads back to _unpolished contigs_...

We need to make a new mapping file and generate depth file from that, **but the mapping is a time consumings step, so we have done this for you before todays session**
You can find the **depth file** in `/mnt/SCRATCH/bio326-21/MetaGenomeBinning/`. Copy this to your `MetaGenomeBinningBio326`. 

Make your way to this directory using `cd MetaGenomeBinningBio326` and

```
cp /mnt/SCRATCH/bio326-21/MetaGenomeBinning/depth.txt .
```
Check that you have the depth file in your directory using `ls`. Which path is this (`pwd`). HINT: this will be your `mapping_path`

You will also need a directory for your binning results, 
``` 
mkdir Binning.dir 
```
HINT: this will be your `out_path`

In this directory, you will also generate your bash job script. I have made a template below that you can copy paste into a new .sh file (e.g. ´nano binning.SLURM.sh`) 
```
#!/bin/bash

#SBATCH -N 1
#SBATCH -J binning
#SBATCH -n 1
#SBATCH --mem=10G
#SBATCH --partition=smallmem


#REMEMBER TO CHANGE THE PATHS   
contig_path='/PATH/TO/POLISHED/CONTIGS'        #CHANGE
mapping_path='/PATH/TO/DEPTH/FILE'              #CHANGE
out_path='/PATH/TO/YOUR/OUTPUT/DIR'             #CHANGE

singularity exec /cvmfs/singularity.galaxyproject.org/m/e/metabat2:2.15--h986a166_1 metabat2 -i $contig_path/polished_ONT-contigs.fasta -a $mapping_path/depth.txt -o $out_path/ONT_bin

#MAGs are given the -o name as prefix, here; ONT-bin

```

or you can copy the bash job file from here: 

```
cp /mnt/SCRATCH/bio326-21/MetaGenomeBinning/binning.SLURM.sh .
```

**REMEMBER, YOU HAVE TO CHANGE YOUR PATHS IN BOTH CASES** and the assembly path is where you have the **polished contigs** from last session! Dont have the polished contigs? No worries, you can pick them up here: 
``` 
cp /mnt/SCRATCH/bio326-21/MetaGenomeAssembly/MetaG_Assembly.dir/polished_ONT-contigs.fasta ./
``` 

Send the job to the queue: 

```
sbatch binning.SLURM.sh
```
and monitor the job using `squeue -u youruserID`

Not in the list? This job will finish in a second - so check your out-path! Empty out-path or error in slurm output file?? **CHECK YOUR PATHS** - are they correct? 


When the binning job is done, you should see something like this in your `Binning.dir`

![image](https://user-images.githubusercontent.com/65181082/115470587-daef9480-a236-11eb-9d60-63ef6f126f30.png)


_How many bins did you get?__
Look at the file size of the bins using `ls -l` (rough estimate to have an idea of the genome size. We will look at the actual genome size in the checkM results below), have you seen similar contig sizes earlier in the this data lab?_

Spend some time exploring the MAGs; 
* How many contigs are in each MAGs? And what can this tell us?
* Where did the circular contigs go? 

_We lost the circular information in the header during the polishing step, but this is how ot looked like:_

![image](https://user-images.githubusercontent.com/65181082/115472538-5f8fe200-a23a-11eb-9193-ba47e4323f53.png)

use e.g. 
```
grep "^>" binID
```
what do you get if you add the parameter `-c`, `grep -c "^>" binID`? 


# QUALITY CHECK

Some of the bins can be considered as high quality MAGs (and this is when the fun starts) while others are just trash bins (_reason_: low coverage, difficult to assemble, similar to others, HGT etc.) 

_This is what I meant with "be critical" during the presentation last week. Binning is powerful, but the bins should be evaluated._ 

To check the quality of the generated bins/MAGs, we will use CheckM: https://github.com/Ecogenomics/CheckM/wiki

![image](https://user-images.githubusercontent.com/65181082/115446311-50e40380-a217-11eb-93e7-9b8befabc672.png)



Modify and start the bash job script **in the same manner as before**

```
#!/bin/bash

#SBATCH -N 1
#SBATCH -J checkM
#SBATCH -n 8
#SBATCH --mem=140G
#SBATCH --partition=hugemem


#REMEMBER TO CHANGE THE PATHS   
bin_path='/PATH/TO/YOUR/BINS' #CHANGE
out_path='/mnt/SCRATCH/bio326-21-0/MetaGenomeBinningBio326/Binning.dir/checkM'  #ONLY CHANGE USER ID; job will generater the checkM dir!

singularity exec /cvmfs/singularity.galaxyproject.org/c/h/checkm-genome:1.1.3--py_1 checkm lineage_wf -x fa $bin_path/ $out_path/

singularity exec /cvmfs/singularity.galaxyproject.org/c/h/checkm-genome:1.1.3--py_1 checkm qa $out_path/lineage.ms $out_path -o 2 > $out_path/ONT_qa_bins

```

or you can copy the bash job file from here: 

```
cp /mnt/SCRATCH/bio326-21/MetaGenomeBinning/checkM.SLURM.sh .
```

**REMEMBER, YOU HAVE TO CHANGE YOUR PATHS IN BOTH CASES** 


Send the job to the queue: 

```
sbatch checkM.SLURM.sh
```
and monitor the job using `squeue -u youruserID`. **NB: This job will take some time to finish!** While you are waiting, head down to **TAXONOMIC CLASSIFICATION**

_When done:_ This job will generate a directory called `checkM` in the Binning.dir path. Make your way to this directory, and look at the `ONT_qa_bins` output. 

```
less -S ONT_qa_bins
```
_should look something like this:_

![image](https://user-images.githubusercontent.com/65181082/115537172-d65cc780-a29a-11eb-9fc7-1238611cdda1.png)


How does it look? Are the MAGs of OK quality?


# TAXONOMIC CLASSIFICATION

It can also be usefull to have an idea of _who_ you have in your sample. For this we will use GTDBtk: https://github.com/Ecogenomics/GTDBTk

We will start this job slightly different that the previous sessions, but the prinsiple is the same. In this case, the bash job script is consstructed so that spesify your input_path when you start the job: 

EXAMPLE: 
```
sbatch gtdbk.classifywf.SLURM.sh PATH/TO/MAGs fasta_files_extension
```
First you will need to make a new directory for this: 

```
mkdir Taxonomy
```

Then copy the `gtdbk.classifywf.SLURM.sh` to your /mnt/SCRATCH/bio326-21-0/MetaGenomeBinningBio326/Taxonomy

```
cd /mnt/SCRATCH/bio326-21-0/MetaGenomeBinningBio326/Taxonomy
```

```
cp /mnt/SCRATCH/bio326-21/MetaGenomeBinning/gtdbk.classifywf.SLURM.sh ./
```
_you do **NOT** have to change the `.sh` script now, run like example above_
```
sbatch gtdbk.classifywf.SLURM.sh /PATH/TO/YOUR/MAGs fa
```

_Is it running?_

GTDBtk may run for some hours, so we will just let it run until next session (Friday) 

**ON FRIDAY YOU WILL DO FUNCTIONAL ANNOTATION OF THE MAGS USING DRAM, AND YOU WILL NEED THE OUTPUT FROM CHECKM AND GTDBTK!**

DRAM: https://github.com/shafferm/DRAM


Thank you for today, we got MAGs yey!








