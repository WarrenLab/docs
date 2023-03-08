# Quick intro to running nextflow pipelines

Many of the repositories in the WarrenLab group are written as [nextflow][nf] pipelines. Rather than repeat the same boilerplate about how to use nextflow in the README for each of these repositories, I'm putting this information here in a central place.

## Installing

Generally, the only thing you need to install to use a nextflow pipeline is the nextflow binary itself. To do this, just run
`curl -s https://get.nextflow.io | bash` and put the binary it creates somewhere in your path.

You **do not** need to clone the repository containing the pipeline or otherwise download the code it contains to run it. Nextflow does that for you.

## Running

Nextflow separates the logic of the pipeline — i.e., what programs to run in what order — from the details about how jobs should be run, such as: whether to run them locally, on a cluster, or in the cloud; what resources to allocate for them; whether to run them inside of a container or conda environment or just use programs that are already installed on your system. This is nice because it means that you can run the same pipeline on different systems just by editing some configuration.

The pipelines here were all developed to run on the Lewis cluster at MU. If you are trying to run them on lewis, hopefully you shouldn't have to do any configuration at all. You can just add `-profile lewis` (note only one dash!) to the command-line arguments and it will use the configuration set up for lewis. Generally, this means:
* using the `BioCompute` partition
* charging the `warrenlab` account
* asking for 48 hours for each job, except where the job has a tendency to take longer, in which case it uses the `long` QOS and asks for 7 days
* using a RAM amount and number of cores per task that makes sense based on the specifications of BioCompute nodes
* if there is a public Docker image available that has everything the pipeline needs, using Singularity to run jobs inside of a container based on it
* if all the dependencies can be easily installed with conda, running everything inside a pre-built conda environment stored on Lewis

If you need to do something different, you can make a copy of the file `nextflow.config` in the repository, edit it to make it work for you, put it in the directory from where you'll be running the pipeline, and then add `-c nextflow.config` to the command-line arguments to nextflow. Nextflow supports all sorts of clusters and cloud systems, so go read the [docs][nf-config] to see how you can configure it to run our pipelines!

## Tips and tricks

### Resuming a run
If a run fails but you don't want to start it over again from the beginning, you can fix the mistake and then run the nextflow command again with the `-resume` flag to avoid redoing completed jobs.

### Investigating problems
For every job, nextflow makes a folder containing:
* `.command.sh`: the commands that got run in this job
* `.command.err|out`: the STDERR and STDOUT, respectively, of the job
* `.command.run`: a wrapper for `.command.sh` that deals with any necessary environment setup like running the command in a container or conda environment. If you are using, e.g., SLURM, this is the actual script that gets submitted to the cluster.
* Links to all the input files for this job
* Any output files the job created

These folders are stored with randomly generated names in your work directory (see below). You can find the folder for a given job by looking for the hash like `[e8/406d52]` next to the job name in the nextflow output, or in the log at `.nextflow.log`. They are extremely useful for figuring out what went wrong with a run.

### Filesystems
Nextflow keeps intermediate files in a work directory, which is by default a subdirectory it creates called `work/` wherever it is run. There is often a whole lot of IO happening in this directory, so it is good if the work directory is on a high-performance storage system. However, nextflow needs to be able to create locks on files in the directory where it is being run, which some high-performance filesystems do not support. This is a problem on lewis, where HPC is faster than HTC but does not support locks. To get around this, you can set the `$NXF_WORK` environment variable to point to somewhere on a fast filesystem, but then run the `nextflow` command from a filesystem that supports locks. On lewis, this could looks like:

```bash
export NXF_WORK=/storage/hpc/group/[lab]/users/[user]/nextflow_work
mkdir -p $NXF_WORK
cd /storage/htc/[lab]/users/[user]/project1/
nextflow run ...
```

[nf]: <https://www.nextflow.io/>
[nf-config]: <https://www.nextflow.io/docs/latest/config.html>
