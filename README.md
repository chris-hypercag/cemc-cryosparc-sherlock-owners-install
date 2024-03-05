# Installing CryoSPARC in Sherlock Owner's Partition
The following instructions are intended for Sherlock users who have permission to run the CryoSPARC GUI from the Owners partition, and to easily queue jobs in any partition they have permission to access. 

Attribution: These instructions for installing CryoSPARC on Sherlock are adapted from instructions developed and published by [jnoh2](https://github.com/jnoh2/cryosparc-install/blob/main/README.md). The instructions were originally derived from [CryoSPARC's](https://guide.cryosparc.com/setup-configuration-and-management/how-to-download-install-and-configure/downloading-and-installing-cryosparc).
### Table of Contents
* [Manual Installation Steps](https://github.com/chris-hypercag/cemc-cryosparc-sherlock-install/blob/main/README.md#manual-installation-steps)
    * [Step 0: Before Installing](https://github.com/chris-hypercag/cemc-cryosparc-sherlock-install/blob/main/README.md#step-0-before-installing)
    * [Step 1: Download CryoSPARC](https://github.com/chris-hypercag/cemc-cryosparc-sherlock-install/blob/main/README.md#step-1-download-cryosparc)
    * [Step 2: Install CryoSPARC](https://github.com/chris-hypercag/cemc-cryosparc-sherlock-install/blob/main/README.md#step-2-install-cryosparc)
    * [Step 3: Create Submission Scripts](https://github.com/chris-hypercag/cemc-cryosparc-sherlock-install/blob/main/README.md#step-3-create-submission-scripts)
    * [Step 4: Connect to the CryoSPARC GUI](https://github.com/chris-hypercag/cemc-cryosparc-sherlock-install/blob/main/README.md#step-4-connect-to-the-cryosparc-gui)
    * [Step 5: Configure CryoSPARC](https://github.com/chris-hypercag/cemc-cryosparc-sherlock-install/blob/main/README.md#step-5-configure-cryosparc)
    * [Step 6: Clean Up](https://github.com/chris-hypercag/cemc-cryosparc-sherlock-install/blob/main/README.md#step-6-clean-up)
* [Starting the CryoSPARC GUI after Installation](https://github.com/chris-hypercag/cemc-cryosparc-sherlock-install/blob/main/README.md#starting-the-cryosparc-gui-after-installation)
    * [Connect to the CryoSPARC GUI](https://github.com/chris-hypercag/cemc-cryosparc-sherlock-install/blob/main/README.md#connect-to-the-cryosparc-gui)
    * [Submit Jobs](https://github.com/chris-hypercag/cemc-cryosparc-sherlock-install/blob/main/README.md#submit-jobs)
* [Adding Additional Parameters for the Submission Script](https://github.com/chris-hypercag/cemc-cryosparc-sherlock-install/blob/main/README.md#adding-additional-parameters-for-the-submission-script)
    * [Step 1: Name the variable for you want to modify](https://github.com/chris-hypercag/cemc-cryosparc-sherlock-install/blob/main/README.md#step-1-name-the-variable-for-you-want-to-modify)
    * [Step 2: Edit your submission script](https://github.com/chris-hypercag/cemc-cryosparc-sherlock-install/blob/main/README.md#step-2-edit-your-submission-script)
    * [Step 3: Connect the new job submission script to CryoSPARC](https://github.com/chris-hypercag/cemc-cryosparc-sherlock-install/blob/main/README.md#step-3-connect-the-new-job-submission-script-to-cryosparc)
    * [Step 4: Indicate the use of the parameter on the CryoSPARC GUI](https://github.com/chris-hypercag/cemc-cryosparc-sherlock-install/blob/main/README.md#step-4-indicate-the-use-of-the-parameter-on-the-cryosparc-gui)
         
## Manual Installation Steps
### Step 0: Before Installing
Before starting the install, you need to obtain a CryoSPARC license. Fill out the download form on CryoSPARC's [website](https://cryosparc.com/download), and select the option that best describes your use case. For most Sherlock users the "I am an academic user carrying out non-profit academic reasearch at a university or educational/research." option will suffice. CryoSPARC will send you an email containing the license number within 24 hours.

### Step 1: Download CryoSPARC
Log on to Sherlock from a terminal window. Once logged in start an interactive job session and replace \<group-name\> with your PI's Sherlock group name:
```
sdev -t 2:00:00 -p <group-name> -g 1
```
Next, set several environment variables that will be used throughout the installation. Depending on where you want to install CryoSPARC you may want to replace $GROUP_HOME with $OAK.
```
export SUNETID=$USER
export CS_PATH=$GROUP_HOME/$USER/cryosparc/4.4.1
```
Next, pick a five-digit number ending in 0 between 49160 and 65530, that does not conflict with your fellow lab members, and use it as your \<PORTNum\>. The port number tells CryoSPARC where to output the GUI when creating an ssh tunnel from your browser.
```
export PORT_NUM=<PORTNum>
```
Next, set a license environment variable by replacing \<LicenseID\> with the number emailed to you in Step 0. 
```
export LICENSE_ID=<LicenseID>
```
Create the install and database directories using the $CS_PATH variable defined above,
```
mkdir -p $CS_PATH
mkdir -p $CS_PATH/cryosparc_db
cd $CS_PATH
```
Download the compressed files (.tar.gz) for the CryoSPARC master and worker programs
```
curl -L https://get.cryosparc.com/download/master-latest/$LICENSE_ID -o cryosparc_master.tar.gz
curl -L https://get.cryosparc.com/download/worker-latest/$LICENSE_ID -o cryosparc_worker.tar.gz
```
Decompress the files
```
tar -xvf cryosparc_master.tar.gz cryosparc_master
tar -xvf cryosparc_worker.tar.gz cryosparc_worker
```
### Step 2: Install CryoSPARC
Install CryoSPARC Master instance
```
cd $CS_PATH/cryosparc_master
./install.sh --license $LICENSE_ID --dbpath $CS_PATH/cryosparc_db --port $PORT_NUM --yes
```
When install is complete, the file `config.sh` is created. You now have two choices, to run the CryoSPARC master from a dedicated node or from any node in your group partition. To run from a dedicated node, first find the hostname of the node your goup is running from. In your prefered text editor (i.e. vim, nano), open `config.sh`, and modify the third line `export CRYOSPARC_MASTER_HOSTNAME="sh##-##n##.int"` to look like this: `export CRYOSPARC_MASTER_HOSTNAME=<hostname>` where \<hostname\> is the dedicated node hostname. To genearlize CryoSPARC to run from any group partition node, open `config.sh` in your prefered text editor (i.e. vim, nano), and modify and comment out the third line `export CRYOSPARC_MASTER_HOSTNAME="sh##-##n##.int"` to look like this: `#export CRYOSPARC_MASTER_HOSTNAME=$(hostname)`. Now, below this line add the following line `export CRYOSPARC_FORCE_HOSTNAME=true`. Save the changes and exit the text editor to return to the terminal.

Next, start the CryoSPARC master instance
```
./bin/cryosparcm start
```
Create your CryoSPARC login credentials. Replace each of the five fields with your own details---keep the quotation marks when entering your information but remove the brackets, for example `--email "jane@stanford.edu"`.
```
./bin/cryosparcm createuser --email "<e-mail>" --password "<password>" --username "<username>" --firstname "<firstname>" --lastname "<lastname>"
```
Install CryoSPARC Worker
```
cd $CS_PATH/cryosparc_worker
ml cuda/11.7.1
./install.sh --license $LICENSE_ID --yes
cd $CS_PATH
```
### Step 3: Create Submission Scripts
Now let's set your group name as an environment variable to automate the next steps
```
export GROUPNAME=<group-name>
```
Next, enable the master instance to run jobs on Sherlock. For this you will need the files `cluster_info.json`, `cluster_script.sh`, and `cs-master.sh`. Copy and paste the following code blocks into the terminal. Clicking the copy icon in the upper right hand corner of the code block will insure the entire field is copied. The `cat` command will automatically concatenate the lines in between it and the end-of-file marker (EOF) and pass it to file named `cluster_info.json`. 
```
cat <<EOF >  cluster_info.json
{
    "name" : "Sherlock",
    "worker_bin_path" : "$CS_PATH/cryosparc_worker/bin/cryosparcw",
    "cache_path" : "$L_SCRATCH",
    "cache_reserve_mb" : 10000,
    "cache_quota_mb": 500000,
    "send_cmd_tpl" : "{{ command }}",
    "qsub_cmd_tpl" : "sbatch {{ script_path_abs }}",
    "qstat_cmd_tpl" : "squeue -j {{ cluster_job_id }}",
    "qdel_cmd_tpl" : "scancel {{ cluster_job_id }}",
    "qinfo_cmd_tpl" : "sinfo"
}
EOF

```
Copy and paste the following code block to create `cluster_script.sh`. This script provides a template to CryoSPARC for submitting worker jobs.
```
cat <<EOF >  cluster_script.sh
#!/bin/bash
#
#SBATCH --job-name=cs-{{ project_uid }}-{{ job_uid }}
#SBATCH --output={{ job_log_path_abs }}
#SBATCH --error={{ job_log_path_abs }}
#
#SBATCH --partition={{ partition_requested }}
#SBATCH --nodes=1
#SBATCH --ntasks={{ num_cpu }}
#SBATCH --gpus={{ num_gpu }}
#SBATCH --mem={{ (ram_gb*2)|int }}G
#
#SBATCH --time={{ time_requested }}
#
#SBATCH --mail-type=BEGIN,END,FAIL
#SBATCH --mail-user={{ sunetid }}@stanford.edu

echo "\$(date): job \$SLURM_JOBID starting on \$SLURM_NODELIST"

ml cuda/11.7.1

echo "Starting cryosparc worker job"

{{ run_cmd }}

echo "Finished cryosparc worker job"
EOF

```
Copy and paste the following code block to create `cs-master.sh`. This script will start and restart the CryoSPARC master instance.
```
cat <<EOF >  cs-master.sh
#!/bin/bash
#
#SBATCH --job-name=cs-master
#SBATCH --error=cs-master.err.%j --output=cs-master.out.%j
#
#SBATCH --dependency=singleton
#
#SBATCH --partition=$GROUPNAME
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=2
#SBATCH --mem=16G
#
#SBATCH --time=7-00:00:00
#SBATCH --signal=B:SIGUSR1@360
#
#SBATCH --mail-type=BEGIN,END,FAIL
#SBATCH --mail-user=$SUNETID@stanford.edu

_resubmit() {
    ## Resubmit the job for the next execution
    echo "\$(date): job \$SLURM_JOBID received SIGUSR1 at \$(date), re-submitting"
    cd $CS_PATH
    date -R >> cs-master.log
    ./cryosparc_master/bin/cryosparcm stop >> cs-master.log
    sbatch \$0
}
trap _resubmit SIGUSR1

cd $CS_PATH

echo "Loading cryosparc GUI"

## run cryosparc server and append output to log file
echo >> cs-master.log
date -R >> cs-master.log
./cryosparc_master/bin/cryosparcm restart >> cs-master.log

echo "Loaded  cryosparc GUI"

echo "\$(date): job \$SLURM_JOBID starting on \$SLURM_NODELIST"

## Keep job alive while cryosparc server is up
while true; do
    echo "\$(date): normal execution"
    sleep 300
done
EOF

```
Last step, connect the master with Sherlock cluster information and submission script.
```
./cryosparc_master/bin/cryosparcm cluster connect
```
### Step 4: Connect to the CryoSPARC GUI
Open a separate terminal on your computer (do not log into Sherlock). In the new terminal execute the following command to enable ssh tunneling, replacing sh##-##n## with the hostname of your interactive session, \<PORTNum\> with the five-digit number you selected in Step 1, and \<SUNetID\> with your SUNetID. The hostame can be found in your Sherlock terminal where you enter commands.
```
ssh -NfL localhost:<PORTNum>:sh##-##n##:<PORTNum> <SUNetID>@sherlock.stanford.edu
```
Then from your computer, open any browser and go to the following url, 
```
localhost:<PORTNum>
```
Once you see the login screen, you can log in with the credentials you chose in Step 2.

### Step 5: Configure CryoSPARC
Now we will add Key-Value pairs to CryoSPARC's cluster configuration tab. This will allow CryoSPARC to recognize our user-defined parameters in cluster_script.sh. The Key defines the variable name and Value defines the default value.
1. Once logged in, go to the admin page (key symbol on the left)
2. Go to Cluster Configuration Tab
3. Add a few Key-Value pairs, replacing \<SUNetID\> with your SUNetID
- Key = time_requested | Value = 01:00:00
- Key = partition_requested | Value = normal
- Key = sunetid | Value = \<SUNetID\>

### Step 6: Clean Up 
At this point both the master and worker instances are installed and configured for Sherlock. However, since the master instance is running in a time-limited interactive session, it is not prudent to start a new CryoSPARC project. For that you will need to run the master instance as a Sherlock job. 

To finalize the installation and clean up, go to the terminal window running your interactive session and stop the cryoSPARC master instance and exit sh_dev mode.
```
cd $CS_PATH
./cryosparc_master/bin/cryosparcm stop
exit
```

## Starting the CryoSPARC GUI after Installation
The max runtime for a job on Sherlock 7 days. The `_resubmit()` function in `cs-master.sh` automatically requeues the CryoSPARC master instance when the time limit is reached. If a worker job doesn't finish before the 7 day time limit, the worker job will mostly likely terminate itself.  

It is highly recommended you cancel the master instance each time you are done for the day and resubmit the job when you want to start working again. This helps free up Sherlock resources for other users and keeps your fairshare score from depleting. Your fairshare score is an important metric when running in the normal partition; it effects how long Slurm will hold your job before allocating it resources. The higher your fairshare score, the faster your job will get through the queue. You can prevent unnecessary depletion of your fairshare score by requesting the minimum number of resources (cpus, memory, runtime) needed to run the master and worker jobs.

To submit the master job to the queue, run the following command from your CryoSPARC directory containing `cs-master.sh`:
```
sbatch cs-master.sh
```
To cancel the job when you're done:
```
scancel -n cs-master
```
To check the current status of your job:
```
squeue --me
```
When the master instance is running, `squeue --me` will show an R under ST and will also output the hostname of the master node under NODELIST. The hostname has the format `sh##-##n##`. Copy the hostname for the next step.

### Connect to the CryoSPARC GUI
Now open a separate terminal on your computer. In the new terminal execute the following command to enable port forwarding, replacing sh##-##n## with the hostname, \<PORTNum\> with the port number selected in Step 1, and \<SUNetID\> with your SUNetID, 
```
ssh -NfL localhost:<PORTNum>:sh##-##n##:<PORTNum> <SUNetID>@sherlock.stanford.edu
```
Then on any browser on your computer, go to the following url, 
```
localhost:<PORTNum>
```
Once you see the login screen, you can log in with the credentials you chose in Step 2.
### GUI Not Connecting
Note: If the browser is unable to connect, the port may need to be reset. 
#### Mac and Linux users
From a terminal on your desktop, find the PID number of the open port.
```
lsof -i:<PORTNum>
```
Copy the PID number, and kill the process directly
```
kill <PID>
```
Try rerunning the port forwarding command. If the port did not reset, try closing the browser completely and reopening.
#### Windows users
From a Windows PowerShell on your desktop, find the PID number of the open port.
```
netstat -ano | findstr :<PORTNum>
```
Copy the PID number in the far right column and kill the process directly
```
kill <PID>
```
If the `kill` command returns an error, open your Task Manager, search for a process name ssh, right click and select 'End task.' Now try rerunning the port forwarding command.
### Submit Jobs
For a given job, build your job as needed. When you click "Queue Job," and you're given the option to modify the category "Queue to Lane"
1. Select "Sherlock"
2. Under "Cluster submission script variables" enter the estimated time needed to complete the job, the partition the job will run in, and SUNetID if different from the default.
4. Click "Queue"

Note: The `partition_requested` parameter can be set to any partition on Sherlock. Public partitions include: `normal`, `gpu`, and `bigmem`. Private partitions include the `owners` partition, and PI partions. The current Sherlock cluster submission script is general enough to run from any partition you have permission to access. To see which partitions you have access to, run `sh_part` on Sherlock from the terminal.

## Adding Additional Parameters for the Submission Script
You may want to be able to adjust more parameters in the Sherlock job submission script. 

### Step 1: Name the variable you want to modify
If you want to adjust certain hardcoded sbatch or bash parameters in your submission scripts from within CryoSPARC you can do so by adding Key-Value pairs in the cluster configuation tab. First come up with a unique variable name for the parameter you wish to modify.  For example, the `#SBATCH --partition=` parameter can be modified by declaring a `{{ partition_requested }}` variable name, as seen in `cluster_script.sh`. 

### Step 2: Edit your submission script 
Within your scripts add the new parameter or replace the hardcoded value of a preexisting parameter with your variable name. When using your own variable in scripts, you must keep the curly braces and spaces surrounding the variable name. In CryoSPARC, the curly braces and spacing identify `partition_requested` as a variable.

### Step 3: Connect the new job submission script to CryoSPARC
Make sure you are in the directory that contains `cluster_script.sh` and `cluster_info.json`. Then enter the following:
```
./cryosparc_master/bin/cryosparcm cluster connect
```
### Step 4: Indicate the use of the parameter on the CryoSPARC GUI
1. Go to your CryoSPARC master instance on your browser.
2. Go to admin (key symbol on the left)
3. Go to Cluster Configuration tab
4. Add the Key-Value pair for which the "Key" is your variable name WITHOUT curly braces and spaces (i.e. `partition_requested`), and the "Value" should be the default for your parameter. In this example, you would add the following:
- Key = partition_requested | Value = normal
