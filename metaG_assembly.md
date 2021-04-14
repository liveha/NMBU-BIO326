# **Metagenome assembly**

In this session, we will **start an assembly of a metagenome** from a human gut sample (enrichment). 
The metagenome has been sequences with both ONT minION (long-reads) and with Illumina HiSeq (short reads). In this course, we will assemble the long reads using CANU as we did for the microbe from the salmon gut, followed by polishing the contigs using the short reads. We will use BUSCO to compare the quality of the circular contigs before and after polishing using the short reads (Friday), before we bin the contigs to Metagenome-Assembled Genomes (MAGs) and annotate the MAGs (next week). 

Compared to the Salmon gut microbe, this ONT sequencing only ran for 6 hours on the MinION. Since this DNA is from a more complex microbial community (we expect between 10 and 20 bugs to be present) the number of sequences per microbe, will be lower, meaning _the sequencing coverage is lower_. Your sequence coverage is always something to keep in mind when analysing your data. 

This session will be slightly different than the previouse dry-lab sessions. We will only start one job (assembly), but since you have ran an assembly using canu before, you are encouraged to try a tiny bit more on your own this time. A copy-paste option is available if you scroll, and we are available for questions and breakout rooms.    

Remember that you can always:
* Check where you are (pathway directory) by typing `pwd`
* List content in your directory by typing `ls` and `ls -l`
* Change your directory usind `cd` , e.g. `cd /path/to/where/I/wanna/go/`
* copy files using `cp`, e.g. `cp /path/to/myfile.fastq /path/to/new/directory/` or if you are in the directory you want to move the file to, simply `cp /path/to/file/myfile.fastq .` 

## Obtain the fastq files 

The minKNOW software will by default aliquote and store the fast5 and subsequently the basecalled fastq sequences in chuncks of mulitple sequences based on total size. *These fastq files have already been concatinated to one file using `cat`. The sequences in the concatinated fastq file has then been filtered using `Filtlong`*    

_In brief_: you should make a directory for this metagenome-assembly session and copy the fastq file available here `/mnt/SCRATCH/bio326-21/MetaGenomeAssembly/GutMetagenome_filtered_rawReads.fastq.gz` to your newly generated directory. 
No prob, all done? Great, move to **Assembly**


_Confused_? No worries, peekaboo for copy-paste below:

Log in to Orion, and make your way to SCRATCH: 

```
cd $SCRATCH/
```

Make a directory ("folder") called **MetaGenomeAssemblyBio326** (or something else that make sense for you), 

```
mkdir MetaGenomeAssemblyBio326
```

Enter this directory (`cd MetaGenomeAssemblyBio326`) 
and copy the trimmed and concatinated fastq files to your newly generated directory: 

```
cp /mnt/SCRATCH/bio326-21/MetaGenomeAssembly/GutMetagenome_filtered_rawReads.fastq.gz .
```
you should also make a result directory for your generated assembly files

```
mkdir MetaG_Assembly.dir
```

A good practice (especially when newbie) is to check that you have all directories and files in place before you run a job. Unsure what you have or where you are? Type `ls` or `pwd` to have a look! 

## Assembly

And now, time to start your assembly using CANU (again)! Luckily for us, canu is already available as a module in Orion. But this is often not the reality, and you usually have to make your own slurm files. 

When you search for a software, you will often find documentation like this: 
https://canu.readthedocs.io/en/latest/quick-start.html

**Before you continue, I would like you to use a some minutes to look at the documentation** and reflect on how you could get started if you want to assemble a genome or a metagenome in e.g. 4 months from now (i.e. not when we tell you what to do).

I will now give you two choices:

1) **Build your own slurm script** based on the documentation you just read, and the previouse sessions with Arturo. Think: what kind of data do I have? what kind of information would I want in my outputs? What parameters i are reccomended (metagenome, ONT, slightly shallow sequencing coverage) 

2) Jump to the section below, **Assembly using pre-made slurm**

If you feel confident in what we have done until now, I would strongly andvise you to try on your own first (option 1). For some of you, this will be easypeasy (and you could even try to assemble using Flye for the comparison: https://github.com/fenderglass/Flye/blob/flye/docs/USAGE.md)  
If this course is the first time you are working with HPC and slurm files, and everything is greek, option 2 is completely OK. 



### Assembly using pre-made slurm

You can either copy and paste this script into your terminal, or copy the metagenome_canu.SLURM.sh file from /mnt/SCRATCH/bio326-21/MetaGenomeAssembly to your metaCANU.Assembly.dir folder

In both cases: **REMEMBER TO CHANGE THE PATHS** and give your job a name. You can edit the metagenome_canu.SLURM.sh file using `vi`, `vim`, `nano` or your favorite text editor.

```
#!/bin/bash

#SBATCH --ntasks=6               # 1 core(CPU)
#SBATCH --nodes=1                # Use 1 node
#SBATCH --job-name=your_job_name   # sensible name for the job
#SBATCH --mem=50G                 # Default memory per CPU is 3GB.
#SBATCH --partition=smallmem # smallmem 1-100 GB RAM, hugemem > 100 GB.

#Load your modules:
module --quiet purge  #Reset the modules to the system default
module load canu/1.9-GCCcore-8.3.0-Java-11  #load canu module

#CHANGE PATHS:
inpath='/PATH/TO/MY_FASTQ'
outpath='/PATH/TO/MY/RESULT-DIR' 

#run 
canu -p XDC-ONT -d $outpath/ corOutCoverage=10000 corMinCoverage=0 corMhapSensitivity=normal genomeSize=5m -nanopore $inpath/GutMetagenome_filtered_rawReads.fastq.gz

#-p assembly prefix --> XDC=studyID ONT=sequencing_technology
#-d assembly directory
 
#exit
module unload

```
or 

```
cp  /mnt/SCRATCH/bio326-21/MetaGenomeAssembly/metagenome_canu.SLURM.sh .
```
Start your job: 
```
sbatch metagenome_canu.SLURM.sh
```

Check your slurm job - is it runnning? 

```
squeue -u youruserID
```

Once running, the assembly will take approx 24 hours to finish. We will continue with polishing of the contigs on friday, and hopefully we get some nice circular genomes!!









