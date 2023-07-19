---
layout: post
title: "Using a Script to Submit Multiple Jobs on a Cluster"
date: 2023-07-19 17:05:00 +0100
categories: jekyll update
---
My PhD was on Monte Carlo methods, a family of techniques used for Bayesian inference: inferring the distribution of the unknown parameters in a statistical model. As a part of my research, I wrote a lot of code and ran many computer experiments. As the experiments became more intensive, my laptop was no longer adequate for performing them. I learnt how to compute on a cluster, which is essentially a large number of computers connected together into a network. Computing on a cluster was vital to completing some of the simulation experiments required for my research. This post is about what I learnt about computing on a cluster.

# The form of a Monte Carlo simulation experiment

Suppose that you want to know the probability that a coin toss results in the coin landing on Heads. To estimate this probability, you could flip the coin 100 times, count how many times it lands on Heads then divide this number by 100.  This example provides intution for the core idea of the Monte Carlo method, which really is very simple. We may learn about the expected outcome of a random event by simulating the event many times.

In Bayesian statistics, we are interested in learning about the distribution of the unknown parameters of a statistical model. We sample this distribution multiple times, then use these samples to approximate quantities such as the mean.

In Monte Carlo methodology research, we may have devised a novel way to generate samples from a given distribution. An experiment would then typically consist of the following steps. First, we choose a distribution with a known quantity. For example, suppose that we know the distribution's exact mean. Then:
1. Sample the distribution using the novel method and use the samples to approximate the mean.
2. Compute the distance between the exact and approximate mean.

It's important to understand that the output of a run of a Monte Carlo algorithm is stochatic (random). That is, if a different pseudo-random number generator seed is provided each time, the output of each run of a program implementing a Monte Carlo algorithm will be different. Thus, to understand how well our algorithm approximates the true mean, we'll want to repeat steps 1 and 2 many times (say 100).

Thus in Monte Carlo research, we often want to run the same (expensive) program multiple times, with each instance of the program having a different input (the pseudo-random number generator seed). Suppose that our program has been compiled into an executable `my_executable.out`, which takes as input some text file `my_input.txt` that specifies the pseudo-random number generator seed and other program inputs. Suppose that since our program implements a computationally intensive Monte Carlo method, it takes an hour to run and that we want to run the program 100 times with different inputs. Obviously, doing this on a laptop will take a long time! 

# Using a cluster

Typically a cluster will consist of a submission node, connected to a number of other nodes. A node is a single compute server within the cluster. You log onto the submission node, from where you submit jobs. The submission node allocates your job to the other servers, depending on availability (and some other factors). You may be able to run some short programs directly on the submission node, though you should not run intensive programs on the submission node, since it's primary use is for allocating jobs to other servers. The details of how the submission node works will depend on the specific cluster.

A basic script for submitting a job is as follows. The script is adapted from reference [1]. Suppose that `my_executable.out` is some program:
    
    #!/bin/bash
    #PBS -N my_job
    #PBS -q standard
    
    cd ${HOME}/my_dir
    ./my_executable.out < my_input.txt

The first line is a shell interpreter definition. The second line names your job `my_job`. The third line submits the job to the `standard` queue (there may be different queues, for example for having single or multiple processor cores assigned to the job). The fourth command changes the directory to the relevant directory. Finally, `./my_executable.out < my_input.txt` executables the program. Suppose that this script is called `my_wrapper`. We submit this job using

    qsub my_wrapper

# Using a script to submit multiple jobs on a cluster

The previous section presented a script for submitting a single job to the cluster. But what if want to submit a large number of jobs? We don't want to manually write a script for each, since this would be slow and tedious. Instead, we can use a shell script to write scripts and submit them as jobs!

    #!/bin/bash

    seed=0

    # Create a subdirectory for the input files, if one does not exist already
    if [ ! -d "input" ]
    then
        mkdir input
    fi

    # Create input text files, stored in the input subdirectory
    for i in {1..100}
    do
        printf "${seed}\n" > input/input${i}.txt
        let seed++
    done

    # Create a subdirectory for the scripts that will be submitted as jobs
    if [ ! -d "wrapper" ]
    then
        mkdir wrapper
    fi

    # Create and submit the jobs
    for i in {1..100}
    do
        # Create the script for the job
        printf "#!/bin/bash\n" > wrapper/wrapper${i}
        printf "#PBS -N my_prog${i}\n" >> wrapper/wrapper${i}
        printf "#PBS -q standard\n" >> wrapper/wrapper${i}
        printf "cd ${PWD}\n" >> wrapper/wrapper${i}
        printf "./my_executable.out < input/input${i}.txt\n" >> wrapper/wrapper${i}

        # Submit the job
        qsub wrapper/wrapper${i}
    done

# Closing Remarks

Using a cluster requires more effort, but is well worth it for running multiple, large programs. Editing a program whilst logged-on to the cluster is more difficult, because edits must be made through the terminal window, for example by using vim. In addition, data visualisation must be done on your local computer, so data must be transferred to and from the cluster, which can take a small amount of time. Nonetheless, for large jobs, cluster computing is a game-changer and can help you do a lot more computing.

# References

[1] Information about Imperial College London Mathematic Department's cluster: [https://sysnews.ma.ic.ac.uk/compute-cluster/](https://sysnews.ma.ic.ac.uk/compute-cluster/).