# POLISHING AND EVALUATING ONT CONTIGS

Today we will polish the contigs from the metagenome assembly using short-read sequences form Illumina HiSeq sequencing of the same sample. 
_If we have time, we migth also start a binning job._


## But first, recap on how to submit a job via `sbatch`

**IMPORTANTLY, you will NOT run this job interactively with `srun`. Instead, you will have to generate a bash job script that you submit via `sbatch`**

_Why is this important?_
Well, interactive jobs are good to test your system and check or troubleshoot your script. But what if you are running a job that will take hours, days or even weeks to complete (which is often the case in genome/metagenome sequencing)? For larger computations in Orion, you have to submit the job script into the queue. You can read more about interactive vs bash job scripts here (sigma is another HPC, very similar to Orion): https://documentation.sigma2.no/jobs/submitting.html

When you want to process or analyse your genome sequencing data (genome, metagenome, eukaryote, prokaryote, same-same), you usually have to 
1) find the software you need, 
2) install the software you need and 
3) create a bash job file to run the job. Most of the time, you will have to create these job files on our own. 
Let me give you an example: **We gonna use Racon to polish our contigs** and if we google _Racon_ we will find the Racon Documentation (in addition to fotos of cute trash pandas): https://github.com/isovic/racon

This documentation tells you how to run Racon (a printscreen): 

![image](https://user-images.githubusercontent.com/65181082/114894301-d11cfa00-9e0e-11eb-82f9-9ab678c70f0c.png)

And this is the information you usually have when you to start your job. Although copy-paste can get you far, reality is usually a bit more work than just copy-paste. After todays session, you will hopefully be a step closer to nailing this on your own!  


**To make a job script**, you first have to generate a bash job file, e.g. `myjob.sh`. This can be done via `vi`, `vim` or `nano`. If you are new to text editors, this page might be helpfull https://www.linux.com/topic/desktop/introduction-text-editors-get-know-nano-and-vim/ (NB: my favorite is `vi`, but `nano` is considered to be more intuitive)

EXAMPLE
```
vi myjob.sh
```

This will generate a text file where you can start creating your bash job script.

A basic bash job script will look something like this:

EXAMPLE
```
#!/bin/bash
#SBATCH --ntasks=1               # 1 core(CPU)
#SBATCH --nodes=1                # Use 1 node
#SBATCH --job-name=my_job_name   # sensible name for the job
#SBATCH --mem=1G                 # Default memory per CPU is 3GB.
#SBATCH --partition=smallmem     # Use the smallmem-partition for jobs requiring < 10 GB RAM.

# If you would like to use more please adjust this.

## Below you can put your scripts
# If you want to load module
module load racon

# Paths
inpath='/my/path/to/input-files'
outpath='/my/path/to/result-dir'

# Insert your script here
racon [options ...] <sequences> <overlaps> <target sequences>

```

NB: We have simplified it in this session, and made templates that you can copy from directory or that you can copy paste from this document and into the txt file. The only thing you will need to change when we create the job script for polishing (below) and evaluation is: 
1)  --job-name
2)  #Paths

Once created, you will have to save your script and submit the job via:

EXAMPLE
```
sbatch myjob.sh
```

Remember that you can always:

Check where you are (pathway directory) by typing `pwd` 
List content in your directory by typing `ls` and `ls -l`
Change your directory using `cd` , e.g. `cd /path/to/where/I/wanna/go/`
Copy files using `cp`, e.g. `cp /path/to/myfile.fastq /path/to/new/directory/` or if you are in the directory you want to move the file to, simply `cp /path/to/file/myfile.fastq .` 

## Back to the contigs..

# Evaluate: 

Today, we will contine with the contigs from the assembly on Wednsday. If you succeeded in running the canu-job, and if it finished (without error), you should hopefully find a contigs.fasta file in your `MetaG_Assembly.dir` directory.

First have a look on your jobs in queue: 
```
squeue -u username
```

If you have a job there, see if it is running (ST = R) or if it still pending in queue (ST = PD). This is how mine looked like: 
![image](https://user-images.githubusercontent.com/65181082/114903619-455b9b80-9e17-11eb-969e-a2c583490860.png)

If you started a job successfully, but your job queue is empty, head over to your MetaG_Assembly.dir using `cd` and see if you have any results file using `ls -l`:

```
cd /mnt/SCRATCH/bio326-21-0/MetaGenomeAssemblyBio326/MetaG_Assembly.dir
```
```
ls -l
```
This folder should look something like this: 
![image](https://user-images.githubusercontent.com/65181082/114939546-e3179080-9e40-11eb-9e28-5a1d25c104c7.png)

Orion will also generate a slurm-log file, e.g. `slurm-12943611.out` where the number corresponds to the jobID. Depending on the software, this file can provide details regarding the program, the run or errors. Check using e.g. `more slurm-12943611.out`

_Orion is packed, so there is a high chance that your job is still pending. However, we have 99 reasons problems, but contigs aint one_: You will find a backup of the contigs  here `/mnt/SCRATCH/bio326-21/MetaGenomeAssembly/MetaG_Assembly.dir`. Copy these to your folder: 

```
cp /mnt/SCRATCH/bio326-21/MetaGenomeAssembly/MetaG_Assembly.dir/XDC-ONT.contigs.fasta
``` 

First thing we will do, is to check how many contigs we have (`-c` stands for count): 

```
grep -c "^>" XDC-ONT.contigs.fasta
```

Lets check how many of these that are circular! From the previous genome assembly, we know that Canu will flag contigs that might be circular **suggestCircular=yes**, so we are looking for this pattern: 

```
grep -i "suggestCircular=yes" XDC-ONT.contigs.fasta
```
This should look something like: 

![image](https://user-images.githubusercontent.com/65181082/114944294-997e7400-9e47-11eb-951f-cfb7cd90af6f.png)

Yey! Look at the size, do you think all of them are actual chromosomal genomes? 

We also wanna do is the check the quality using **BUSCO**. You could run BUSCO interactivly as you did before... but for the training, I will encurage you to run BUSCO as a bash job script, like this: 

Remember, **CHANGE PATHS AND JOB ID**:

```
```
How does it look? *From the Salmon gut microbe genome, we know we should not expect high quality.*

# Polish
Lets see if we can improve this with the accurate short reads from an Illumina run!

The Illumina sequneces can be found here in `....`. Please make a directory and copy the Illumina over: 
```
mkdir ... 
```
``` 
cd ....
```
```
cp ....  
```
check that you have the reads in your directory by typing `ls`. 

All set? 

```

