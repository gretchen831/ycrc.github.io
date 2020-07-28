# Checkpointing

DMTCP "Distributed Multithreaded Checkpointing" allows you to easily save the state of your running job and restart it from 
that point.  This can be very useful if your job fails for any number of reasons: it exceeds the time limit, is preempted from scavenge, etc.

DMTCP does not require any changes to your code or recompilation.  It should work on most sequential or multithreaded/multiprocessing programs as is.

``` bash
module load DMTCP
```

## launching your job under DMTCP

For this simple example, we'll use this python script count.py
``` python
import time, sys
i=0
while True:
  print >> sys.stdout, i,
  sys.stdout.flush()
  i+=1
  time.sleep(1)
```

Run the script interactively:

``` bash
$ dmtcp_launch -i 5 python count.py 
```

It will begin printing to the terminal.  In the background, DMTCP will be writing a checkpoint file every 5 seconds.  
Let it count for a while, then kill it with ^c.

If you look in that directory, you'll see several files related to DMTCP. The *.dmtcp file is the checkpoint file.  
To restart the job from the last checkpoint, do:

``` bash
$ dmtcp_restart -i 5 *.dmtcp 
```

In practice, you'll most likely want to use DMTCP to checkpoint batch jobs, rather than interactive sessions.     

## Arranging for your job to automatically restart when preempted

Here is an example job script that will start a job running, periodically checkpoint it, and automatically requeue the
job if it is preempted:


``` bash
#!/bin/bash

#SBATCH -t 30:00
#SBATCH --requeue
#SBATCH --open-mode=append

#edit following line to put the appropriate module
module load DMTCP

cnt=${SLURM_RESTART_COUNT:-0}
echo "SLURM_RESTART_COUNT = $cnt"

dmtcp_coordinator -i 5 --daemon --port 0 --port-file /tmp/port
export DMTCP_COORD_PORT=$(</tmp/port)

if [[ $cnt == 0 ]]
then
    echo "doing launch"
    rm -f *.dmtcp
    dmtcp_launch -j python count.py
 
elif [[ $cnt > 0 ]]; then
    echo "doing restart"
    dmtcp_restart -j *.dmtcp
else
    echo "Failed to restart the job, exit"; exit
fi
```

Launch the job with sbatch, and watch the numbers appear in the slurm*.out file.  
Then, simulate preemption by doing:

``` bash
$ scontrol requeue <jobid>
```

Because the script specified --requeue, the job will be returned to pending.  Slurm automatically sets a "Begin Time" a couple of minutes
in the future, so the job will pend until then, at which point it will
begin running again.  This time the script will invoke dmtcp_restart, and will continue from the checkpoint.  If you look at the output,
you'll see from the numbers that the job backed up to the previous checkpoint and restarted.  
You can requeue the job several times, and each time it will restart from the last checkpoint.

You should be able to adapt this script to your own job by loading any required modules and 
replacing "python count.py" with your program's invocation.

This example is much more advanced that the simple first example. Some notes:

* DMTCP uses a "controller" to manage the checkpointing.  In the simple example, dmtcp_launch transparently started a controller on the
default port 7779.  In this case, we explicitly start a "controller" on a random port and communicate it via an enviroment variable.
This prevents collisions if multiple DMTCP sessions run on the same node.

* The -j flag to dmtcp_launch tells it to join the existing controller.  

* The env var SLURM_RESTART_COUNT is used to determine if this is a restart or not.

## Using DMTCP with a parallel execution
In this example we run NAMD (a molecular dynamics simulation), using 6 threads on 6 cpus.  We also restart automatically on preemption,
as above.


```
#!/bin/bash

#SBATCH -c 6 
#SBATCH -t 30:00
#SBATCH --requeue
#SBATCH --open-mode=append
#SBATCH -C haswell 

#edit following line to put the appropriate module
module load NAMD/2.12-multicore
module load DMTCP

cnt=${SLURM_RESTART_COUNT:-0}
echo "SLURM_RESTARTCOUNT = $cnt"

dmtcp_coordinator -i 90 --daemon --port 0 --port-file /tmp/port
export DMTCP_COORD_HOST=`hostname`
export DMTCP_COORD_PORT=$(</tmp/port)

if [[ $cnt == 0 ]]
then
    echo "doing launch"
    dmtcp_launch namd2 +ppn $SLURM_CPUS_ON_NODE stmv.namd 
 
elif [[ $cnt > 0 ]]; then
    echo "doing restart"
    dmtcp_restart *.dmtcp

else
    echo "Failed to restart the job, exit"; exit
fi
```

## Additional notes

* dmtcp reopens files when recovering from checkpoints, so most file writes should just work.  However, when requeuing jobs as shown above,
you should take care to do `#SBATCH --open-mode=append`
* keep in mind that recovery from checkpoints does implybacking up to the point of the previous checkpoint.  If your program is continuously 
writing output, the output since the last checkpoint will be replicated.  For many programs (like NAMD) the output is really just logging, so this is not a problem.
* by default, dmtcp compresses checkpoint files.  For large files this can take a long time.  You can turn off comporession with `dmtcp_launch --no-gzip`.

* dmtcp creates a convenience restart script called restart_dmtcp_script.sh with every checkpoint.  In theory you can simply call it to restart:
`./restart_dmtcp_script.sh`
rather than 
`restart_dmtcp *.dmtcp`
However, we have found it to be unreliable.  Your mileage may vary.

The above examples just scratch the surface.  For more information:

A DMTCP [quickstart](https://github.com/dmtcp/dmtcp/blob/master/QUICK-START.md) and 
[documentation](http://dmtcp.sourceforge.net/index.html)

A very helpful page at [NERSC](https://docs.nersc.gov/development/checkpoint-restart/dmtcp/)