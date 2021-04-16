# POLISHING AND EVALUATING ONT CONTIGS

Today we will polish the contigs from the metagenome assembly using short-read sequences form Illumina HiSeq sequencing of the same sample. 


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

NB: We have simplified it in this session, and made templates that you can copy to your directory, or that you can copy paste from this document and into the txt file. The only thing you will need to change when we create the job script for **polishing** is: 
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
squeue -u youruserID
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
cp /mnt/SCRATCH/bio326-21/MetaGenomeAssembly/MetaG_Assembly.dir/XDC-ONT.contigs.fasta .
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

We also wanna do is the check the quality using **BUSCO** again. We already know that BUSCO is available as a container in Orion 

`singularity exec /cvmfs/singularity.galaxyproject.org/b/u/busco\:5.0.0--py_1 busco --help`

We will run BUSCO using the script you used in the genome session, which can be found here: `/mnt/SCRATCH/bio326-21/MetaGenomeAssembly`, called `busco.SLURM.sh` **but with a few minor changes - can you spot them?* 

```
#!/bin/bash

## Job name:
#SBATCH --job-name=BUSCOmetagenome
#
## Wall time limit:
#SBATCH --time=02:00:00
#
## Other parameters:
#SBATCH --cpus-per-task 10
#SBATCH --mem=10G
#SBATCH --nodes 1
#SBATCH --partition=smallmem

## Set up job environment:

module --quiet purge  # Reset the modules to the system default

####Do some work:########

## For debuggin it is useful to print some info about the node,CPUs requested and when the job starts...
echo "Hello" $USER
echo "my submit directory is:"
echo $SLURM_SUBMIT_DIR
echo "this is the job:"
echo $SLURM_JOB_ID
echo "I am running on:"
echo $SLURM_NODELIST
echo "I am running with:"
echo $SLURM_CPUS_ON_NODE "cpus"
echo "Today is:"
date

## Copying data to local node for faster computation

cd $TMPDIR

#Check if $USER exists in $TMPDIR

if [[ -d $USER ]]
        then
                echo "$USER exists on $TMPDIR"
        else
                mkdir $USER
fi

echo "copying files to $TMPDIR/$USER/tmpDir_of.$SLURM_JOB_ID"

cd $USER
mkdir tmpDir_of.$SLURM_JOB_ID
cd tmpDir_of.$SLURM_JOB_ID

cp $SLURM_SUBMIT_DIR/*.contigs.fasta .

fasta=$(ls -1|grep .contigs.fasta)

####RUNNING BUSCO####

echo "Busco starts at"
date +%d\ %b\ %T

singularity exec /cvmfs/singularity.galaxyproject.org/b/u/busco\:5.0.0--py_1 busco \
-i $fasta \
-o metagenome_pre-polish.busco \
-m geno \
--auto-lineage-prok \
-c $SLURM_CPUS_ON_NODE
###########Moving results #####################

echo "moving results to" $SLURM_SUBMIT_DIR/

rm *.fasta #Remove fastq files
rm -r busco_downloads #Remove tmp busco databases downloaded

time cp -r * $SLURM_SUBMIT_DIR/  #Copy all results to the submit directory

####Removing tmp dir#####

cd $TMPDIR/$USER/

rm -r tmpDir_of.$SLURM_JOB_ID

echo "I've done at"
date

```

The bash job script can also be copied to your Assembly directory (same directory as you have your contigs): 

```
cp /mnt/SCRATCH/bio326-21/MetaGenomeAssembly/busco.SLURM.sh .
```

Send the job to Orions queue: 

```
sbatch busco.SLURM.sh
```
*From the Salmon gut microbe genome, we know we should not expect high quality. This will run for a while, so we continue to polishing while we wait for the result*

# Polish
Lets see if we can improve this with the accurate short reads from an Illumina run!

The Illumina sequences can be found here `/mnt/SCRATCH/bio326-21/MetaGenomeAssembly/Illumina_rawReads`, use the `ILLUMINA_TrimmedcatPE.fastq` file! Please make a directory and copy the Illumina over: 
```
mkdir Polishing.dir 
```
``` 
cd Polishing.dir
```
```
cp  /mnt/SCRATCH/bio326-21/MetaGenomeAssembly/Illumina_rawReads/ILLUMINA_TrimmedcatPE.fastq .
```
check that you have the reads in your directory by typing `ls`. 

All set? 

The bash job script below will combine two steps/softwares. First `minimap2` will be used to map the Illumina reads to the ONT contig. When that job is done, `Racon` will start polishing. 

You can; Option 1) 
Make a new .sh file using `vi`, `vim` or `nano`, e.g: 
```
vi polishing_canu.SLURM.sh
```
and **copy the bash script below into the new text file, as you did for busco above** :   

```
#!/bin/bash

#SBATCH -N 1
#SBATCH -J polish
#SBATCH -n 4
#SBATCH --mem=100G
#SBATCH --partition=hugemem

module load Miniconda3
source activate /mnt/SCRATCH/bio326-21/GenomeAssembly/condaenvironments/ONPTools/
module load minimap2

#REMEMBER TO CHANGE PATHS
Illumina_path='/mnt/SCRATCH/bio326-21/MetaGenomeAssembly/Illumina_rawReads' #KEEP
contig_path='/mnt/SCRATCH/PATH/TO/YOUR/CONTIGS'  #CHANGE
mapping_path='/mnt/SCRATCH/PATH/TO/YOUR/CONTIGS' #CHANGE, same as above
out_path='/mnt/SCRATCH/PATH/TO/YOUR/Polishing.dir' #CHANGE


#map the short reads (-ax sr) to contig file and generate .sam
minimap2 -ax sr $contig_path/XDC-ONT.contigs.fasta $Illumina_path/ILLUMINA_TrimmedcatPE.fastq > $mapping_path/SR_to_ONT-contig.sam


#polish ONT contig with short reads from Illumina, using racon
racon --threads 4 --include-unpolished $Illumina_path/ILLUMINA_TrimmedcatPE.fastq $mapping_path/SR_to_ONT-contig.sam $contig_path/XDC-ONT.contigs.fasta > $out_path/polished_ONT-contigs.fasta

conda deactivate
module unload
```

Or you can 
2) copy the bash file calles `polishing_canu.SLURM.sh` from /mnt/SCRATCH/bio326-21/MetaGenomeAssembly to your directory (you should be in a directory called `Polishing.dir`now. No panic if you are not, just keep track on your paths!: 

```
cp /mnt/SCRATCH/bio326-21/MetaGenomeAssembly/polishing_canu.SLURM.sh .
```

**REMEMBER TO CHANGE THE PATHS AND THE JOB NAME IN BOTH CASES**

Ready? 
```
sbatch polishing_canu.SLURM.sh
```
This job takes around ~10-15 min to finish. Monitor the job using `squeue -u youruserID`

When polishing job is finished, we will evaluate the quality of the polished contigs using **BUSCO**, again.
For this, **will use the same BUSCO bash script as above**. First we copy this to our polishing directory: 

```
cp /mnt/SCRATCH/bio326-21-0/MetaGenomeAssemblyBio326/MetaG_Assembly.dir/busco.SLURM.sh ./ 
```

**BUT you will have to change some stuff first..** HINT: When you started the first BUSCO job today, I asked you if you could spot a difference. Could you?   

```
nano busco.SLURM.sh 
```

When changes are made, we can start the job: 
```
sbatch busco.SLURM.sh
```
This will take a little while again.. Perfect time for a short break.

When the job is done: Did the quality of the contigs improved after polishing with short reads? 

The end, happy Friyey!
