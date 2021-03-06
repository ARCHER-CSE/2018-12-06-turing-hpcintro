---
title: "Scheduling jobs"
teaching: 45
exercises: 30
questions:
- "What is a scheduler and why are they used?"
- "How do we submit a job?"
objectives:
- "Submit a job and have it complete successfully."
- "Understand how to make resource requests."
- "Submit an interactive job."
keypoints:
- "The scheduler handles how compute resources are shared between users."
- "Everything you do should be run through the scheduler."
- "A job is just a shell script."
- "If in doubt, request more resources than you will need."
---

An HPC system might have thousands of nodes and thousands of users. How do we decide who gets what
and when? How do we ensure that a task is run with the resources it needs? This job is handled by a
special piece of software called the scheduler. On an HPC system, the scheduler manages which jobs
run where and when.

> ## Job scheduling roleplay (optional)
> 
> Your instructor will divide you into groups taking on different roles in the cluster (users,
> compute nodes and the scheduler). Follow their instructions as they lead you through this
> exercise. You will be emulating how a job scheduling system works on the cluster.
> 
> [*notes for the instructor here*](../guide)
{: .challenge}

The scheduler used in this lesson is {{ site.workshop_sched_name }}. Although {{ site.workshop_sched_name }}
is not used everywhere, running jobs is quite similar regardless of what software is being used. The exact
syntax might change, but the concepts remain the same.

## Running a batch job

The most basic use of the scheduler is to run a command non-interactively. Any command (or series of
commands) that you want to run on the cluster is called a *job*, and the process of using a
scheduler to run the job is called *batch job submission*.

In this case, the job we want to run is just a shell script. Let's create a demo shell script to run
as a test.

> ## Creating our test job
> 
> Using your favourite text editor, create the following script and run it. Does it run on the
> cluster or just our login node?
>
>```
> #!/bin/bash
>
> echo 'This script is running on:'
> hostname
> sleep 120
> ```
> {: .bash}
{: .challenge}

If you completed the previous challenge successfully, you probably realise that there is a
distinction between running the job through the scheduler and just "running it". To submit this job
to the scheduler, we use the `{{ site.workshop_sched_submit }}` command.

```
{{ site.workshop_host_prompt }} {{ site.workshop_sched_submitopt }} example-job.sh
```
{: .bash}
```
{% include /snippets/13/submit_output.{{ site.workshop_sched_id }} %}
```
{: .output}

And that's all we need to do to submit a job. Our work is done -- now the scheduler takes over and
tries to run the job for us. While the job is waiting to run, it goes into a list of jobs called the
*queue*. To check on our job's status, we check the queue using the command `{{ site.workshop_sched_statu }}`.

```
{{ site.workshop_host_prompt }} {{ site.workshop_sched_statu }}
```
{: .bash}
```
{% include /snippets/13/statu_output.{{ site.workshop_sched_id }} %}
```
{: .output}

We can see all the details of our job, most importantly that it is in the "R" or "RUNNING" state.
Sometimes our jobs might need to wait in a queue or have an error. The best way to check
our job's status is with `{{ site.workshop_sched_stat }}`. Of course, running
`{{ site.workshop_sched_stat }}` repeatedly to check on things can be
a little tiresome. To see a real-time view of our jobs, we can use the `watch` command. `watch`
reruns a given command at 2-second intervals. Let's try using it to monitor another job.

```
{{ site.workshop_host_prompt }} {{ site.workshop_sched_submitopt }} example-job.sh
{{ site.workshop_host_prompt }} watch {{ site.workshop_sched_statu }}
```
{: .bash}

You should see an auto-updating display of your job's status. When it finishes, it will disappear
from the queue. Press `Ctrl-C` when you want to stop the `watch` command.

## Customising a job

The job we just ran used all of the scheduler's default options. In a real-world scenario, that's
probably not what we want. The default options represent a reasonable minimum. Chances are, we will
need more cores, more memory, more time, among other special considerations. To get access to these
resources we must customise our job script.

Comments in UNIX (denoted by `#`) are typically ignored. But there are exceptions. For instance the
special `#!` comment at the beginning of scripts specifies what program should be used to run it
(typically `/bin/bash`). Schedulers like {{ site.workshop_sched_name }} also have a special comment
used to denote special scheduler-specific options. Though these comments differ from scheduler to
scheduler, {{ site.workshop_sched_name }}'s special comment is `{{ site.workshop_sched_comment }}`.
Anything following the `{{ site.workshop_sched_comment }}` comment is interpreted as an
instruction to the scheduler.

Let's illustrate this by example. By default, a job's name is the name of the script, but the
`{{ site.workshop_sched_nameopt }}` option can be used to change the name of a job.

Submit the following job (`{{ site.workshop_sched_submitopt }} example-job.sh`):

```
#!/bin/bash
{{ site.workshop_sched_comment }} {{ site.workshop_sched_nameopt }} new_name

echo 'This script is running on:'
hostname
sleep 120
```

```
{{ site.workshop_host_prompt }} {{ site.workshop_sched_statu }}
```
{: .bash}
```
{% include /snippets/13/statu_name_output.{{ site.workshop_sched_id }} %}
```
{: .output}

Fantastic, we've successfully changed the name of our job!

> ## Setting up email notifications
> 
> Jobs on an HPC system might run for days or even weeks. We probably have better things to do than
> constantly check on the status of our job with `{{ site.workshop_sched_stat }}`. Looking at the
> man page for `{{ site.workshop_sched_stat }}`, can you set up our test job to send you an email
> when it finishes?
>
{: .challenge}

### Resource requests

But what about more important changes, such as the number of nodes, cores and memory for our jobs? One thing
that is absolutely critical when working on an HPC system is specifying the resources required to
run a job. This allows the scheduler to find the right time and place to schedule our job. If you do
not specify requirements (such as the amount of time you need), you will likely be stuck with your
site's default resources, which is probably not what we want.

The following are several key resource requests:

{% include /snippets/13/stat_options.{{ site.workshop_sched_id }} %}

Note that just *requesting* these resources does not make your job run faster! We'll talk more about
how to make sure that you're using resources effectively in a later episode of this lesson.

> ## Submitting resource requests
>
> Submit a job that will use 1 full node and 5 minutes of walltime.
{: .challenge}

{% include /snippets/13/env_challenge.{{ site.workshop_sched_id }} %}

Resource requests are typically binding. If you exceed them, your job will be killed. Let's use
walltime as an example. We will request 30 seconds of walltime, and attempt to run a job for two
minutes.

```
{% include /snippets/13/long_job.{{ site.workshop_sched_id }} %}
```

Submit the job and wait for it to finish. Once it is has finished, check the log file.

```
{{ site.workshop_host_prompt }} {{ site.workshop_sched_submitopt }} example-job.sh
{{ site.workshop_host_prompt }} watch {{ site.workshop_sched_statu }}
{% include /snippets/13/long_job_cat.{{ site.workshop_sched_id }} %}
```
{: .bash}
```
{% include /snippets/13/long_job_err.{{ site.workshop_sched_id }} %}
```
{: .output}

Our job was killed for exceeding the amount of resources it requested. Although this appears harsh,
this is actually a feature. Strict adherence to resource requests allows the scheduler to find the
best possible place for your jobs. Even more importantly, it ensures that another user cannot use
more resources than they've been given. If another user messes up and accidentally attempts to use
all of the CPUs or memory on a node, {{ site.workshop_sched_name }} will either restrain their job to the requested resources
or kill the job outright. Other jobs on the node will be unaffected. This means that one user cannot
mess up the experience of others, the only jobs affected by a mistake in scheduling will be their
own.

## Cancelling a job

Sometimes we'll make a mistake and need to cancel a job. This can be done with the `{{ site.workshop_sched_del }}`
command. Let's submit a job and then cancel it using its job number (remember to change the 
walltime so that it runs long enough for you to cancel it before it is killed!).

```
{{ site.workshop_host_prompt }} {{ site.workshop_sched_submitopt }} example-job.sh
{{ site.workshop_host_prompt }} {{ site.workshop_sched_statu }}
```
{: .bash}
```
{% include /snippets/13/del_job_output1.{{ site.workshop_sched_id }} %}
```
{: .output}

Now cancel the job with it's job number. Absence of any job info indicates that the job has been
successfully cancelled.

```
{{ site.workshop_host_prompt }} {{ site.workshop_sched_del }} 38759
... Note that it might take a minute for the job to disappear from the queue ...
{{ site.workshop_host_prompt }} {{ site.workshop_sched_statu }}
```
{: .bash}
```
{% include /snippets/13/del_job_output2.{{ site.workshop_sched_id }} %}
```
{: .output}

## Other types of jobs

Up to this point, we've focused on running jobs in batch mode. {{ site.workshop_sched_name }}
also provides the ability to start an interactive session.

There are very frequently tasks that need to be done interactively. Creating an entire job
script might be overkill, but the amount of resources required is too much for a login node to
handle. A good example of this might be building a genome index for alignment with a tool like
[HISAT2](https://ccb.jhu.edu/software/hisat2/index.shtml). Fortunately, we can run these types of
tasks as a one-off with `{{ site.workshop_sched_submiti }}`.

`{{ site.workshop_sched_submiti }}` (with the right options) can submit a job and then wait for it to start so we can
use the compute node resources interactively. Let's demonstrate this by submitting an
interactive job that uses a single core:

```
{{ site.workshop_host_prompt }} {{ site.workshop_sched_submitiopt }}
```
{: .bash}

You should be presented with a bash prompt. Note that the prompt will likely change to reflect your
new location, in this case the worker node we are logged on. You can also verify this with
`hostname`.

When you are done with the interactive job, type `exit` to quit your session.

{% include links.md %}
