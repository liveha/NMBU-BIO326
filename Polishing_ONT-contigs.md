# POLISHING AND EVALUATING ONT CONTIGS #

## But first, recap on how to submit a job via `sbatch`


_Why is this important??_

When you want to process or analyse your genome sequencing data (genome, metagenome, eukaryote, prokaryote, same-same), you usually have to 1) find the software you need, 2) install the software you need and 3) create a bash job file to run the job. Most of the time, you will have to create these job files from scratch. 
Let me give you an example: **we gonna use Racon to polish our contigs**. If we google _Racon_ we will find the Racon Documentation (in addition to cute trash pandas): https://github.com/isovic/racon

This documentation tells you hao to run Racon (a screen print): 

![image](https://user-images.githubusercontent.com/65181082/114894301-d11cfa00-9e0e-11eb-82f9-9ab678c70f0c.png)

And this is the information you get to start your job. Reality is a bit more work than just copy-paste (although copy-paste can get you a long way), and after todays session, you will hopefully be a step closer to nailing this on your own!  



## Back to polishing.. 
Today, we will contine with the contigs from the assembly on Wednsday. If you succeeded in running the canu-job, and if it finished (without error), you should see this in your `..... ` directory

First have a look on your jobs in queue: 
```
squeue -u username
```

If you have a job there, is it running (ST = R) or is it still pending in queue (ST = PD)?


Orion is packed, so there is a high chance that your job is still pending. If that is 
