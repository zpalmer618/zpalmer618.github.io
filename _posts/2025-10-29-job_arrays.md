---
layout: post
title: A Foray into Slurm Job Arrays
---

One of my research projects as a postdoc at MS&T involves benchmarking a plethora of computational methods and basis sets against numerous molecular systems. At some point, I decided it was reasonably achievable to compute quantum mechanical properties for a combination of 80 systems, 22 methods, and 10 basis sets. That's *a lot* of jobs to submit to a workload manager, like Slurm, all at once!

During my graduate career at Ole Miss, I never really considered that to be a problem and would often submit massive numbers of jobs to the HPC cluster without much thought. At the time, I was working on simulating infrared and microwave spectral data, and doing so necessitated a large number of calculations to be submitted all at once, clogging my own queue. This changed when my graduate school colleague and friend [Brent][brent] introduced his automated [push-button QFF][pbqff](PBQFF) framework. It allowed me to submit a single job that managed all computations, collected data, and generated the simulated spectra. 

For my current project, initially, I almost reverted back to my old habit of submitting large job chunks, wasting most of my queue space. However, reminiscing on my graduate career reminded me that figuring out a way to automate my current situation might be worthwhile. I don't really have the time, and honestly the patience, to manually submit and wait on roughly 17,000 computations. That’s when I discovered [Slurm Job Arrays][job array] in the MS&T HPC cluster docs. This functionality lets you run the same script/function/program on multiple files efficiently. Now, admittedly, I didn't quite understand how to implement job arrays for my current workflow. However, after reading about job arrays from a bunch of sources ***and*** chatting with a graduate student in my group at Mizzou, I was able to successfully implement job arrays for my workflow.

Here’s a modified Slurm submit script with job array implementation:

```bash
#!/bin/bash
#SBATCH --job-name=JobArray
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --array=1-100%10
#SBATCH --cpus-per-task=1
#SBATCH --time=1:00:00 
#SBATCH --mem=9G
#SBATCH --output=./log_files/Job-%a.out

if [ -d /local/scratch/$USER/ ]
        then :
        else mkdir /local/scratch/$USER/
fi

mkdir /local/scratch/$USER/${SLURM_JOBID}_${SLURM_ARRAY_TASK_ID}
cp /home/$USER/job_array_tutorial/Job-${SLURM_ARRAY_TASK_ID}.com /local/scratch/$USER/${SLURM_JOBID}_${SLURM_ARRAY_TASK_ID}/
cd /local/scratch/$USER/${SLURM_JOBID}_${SLURM_ARRAY_TASK_ID}

program.sh Job-${SLURM_ARRAY_TASK_ID}.com

cd /local/scratch/$USER/${SLURM_JOBID}_${SLURM_ARRAY_TASK_ID}

cp /local/scratch/$USER/${SLURM_JOBID}_${SLURM_ARRAY_TASK_ID}/*.out /home/$USER/job_array_tutorial/out_files/
cp /local/scratch/$USER/${SLURM_JOBID}_${SLURM_ARRAY_TASK_ID}/*.chk /home/$USER/job_array_tutorial/out_files/Job-${SLURM_ARRAY_TASK_ID}.chk

rm /local/scratch/$USER/${SLURM_JOBID}_${SLURM_ARRAY_TASK_ID}
```

The `#SBATCH --array` line tells Slurm to handle jobs in array format. For example, `#SBATCH --array=1-100` submits jobs `Job-1.com` to `Job-100.com`, with the assumption that each of the input files you are working with has the same `Job-` prefix. In my actual workflow, I break things down by basis set, so this would change to "631G," for example. Just leaving the job array there, submitting all 100 jobs to the queue, isn't terribly impressive. I *could* have just used the same script that made the input files to also make the individual submit scripts, and then make a `sub_all.sh` script. That's why we also include the `%10.` The `%10` ensures only 10 jobs run simultaneously, maintaining queue space for other projects. That's the crux of the whole thing, really. Lastly, each job uses the same resource limits (`nodes`, `ntasks`, `cpus-per-task`, etc.), applied individually rather than cumulatively. So, for the 100 jobs in the above example, each of them will get one node, one task, one cpu-per-task, etc. No need to pile on the resources.

That's really it. When I was first looking at it, it seemed a whole lot more complicated than what it really turned out to be. The docs were definitely confusing at first, but taking some time to read more about it and chat with someone who had some working experience, I was able to get my own job arrays set up to run in no time. I hope this walkthrough can do the same for you! Whether you've never heard about job arrays before now, or you just want to solidify your understanding. Anyway, I hope this helped! Thanks, folks.

[brent]: https://bwestbro.com
[pbqff]: https://github.com/ntBre/pbqff
[job array]: https://slurm.schedmd.com/job_array.html