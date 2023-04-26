# Reproduce All Experiment Results

In this folder, we provide code and scripts for reproducing figures in our paper.
The name of each script corresponds to the number of each figure in the accepted version of our paper (on the artifact submission site).
Since some of the scripts in this directory take a long computing time (as we mark below), we strongly recommend you create a ***tmux session*** on each node to avoid script interruption due to network instability.


## Full YCSB workloads
* In our paper, we use YCSB workloads, each of which includes 60 million entries and operations.

* The following commands generate all the workloads that we need to reproduce all the results:
    ```shell
    cd SMART/ycsb
    # This takes about 4 hours and 15 minutes. Let it run and you can do your own things.
    sh generate_full_workloads.sh
    ```

## Source Code of Baselines
* For most figures, there are three lines: **SMART**, **Sherman**, and **ART**.
SMART stands for our system. Sherman is [a recent system](https://github.com/thustorage/Sherman) published in *SIGMOG' 22*; we use it as a baseline to show the performance bottleneck of the B+ tree on disaggregated memory (DM). ART is a naive adaptive radix tree (ART) design that we port to DM.

* SMART has been cloned in the home directory of each node. The source code of ART and any other SMART variants is included inside SMART (*i.e.*, this repo).

* We have forked Sherman and modified it to fit our reproduction scripts. Use the following command to clone the [forked Sherman](https://github.com/River861/Sherman) in the same path of SMART (*i.e.*, the home directory) in each node:
    ```shell
    git clone https://github.com/River861/Sherman.git
    ```

## Additional Setup

* Just skip this step if you are using our temporary account. Otherwise, change the `home_dir` value in `./params/common.json` to your actual home directory path (*i.e.*, /users/XXX).

    ```json
    "home_dir" : "/your/home/directory"
    ```


## Start to Run

* In the r650 cluster, the node with ip `10.10.1.1` (with host name `node-1`) can directly establish SSH connections to other nodes. Thus we define it as the **master** node and run our scripts on it. Each script can reproduce the corresponding results and figures in our paper.

    You can run all the scripts with a single batch script on the master node using the following command:
    ```shell
    cd SMART/exp
    # This takes about 17 hours and 25 minutes. Let it run and just check the results in the next day.
    sh run_all.sh
    ```
    Or, you can run the scripts one by one, or run some specific scripts if some figures are skipped or show unexpected results during `run_all.sh` (due to network instability, which happens sometimes):
    ```shell
    cd SMART/exp
    # This takes about 1 hour and 41 minutes
    python3 fig_3a.py
    # This takes about 34 minutes
    python3 fig_3b.py
    # This takes about 33 minutes
    python3 fig_3c.py
    # This takes about 1 hour and 2 minutes
    python3 fig_3d.py
    # This takes about 1 hour and 19 minutes
    python3 fig_4a.py
    # This takes about 5 minutes
    python3 fig_4b.py
    # This takes about 7 minutes
    python3 fig_4c.py
    # This takes about 6 minutes
    python3 fig_4d.py
    # This takes about 4 hours and 20 minutes
    python3 fig_10.py
    # This takes about 4 hours and 24 minutes
    python3 fig_11.py
    # This takes about 27 minutes
    python3 fig_12.py
    # This takes about 17 minutes
    python3 fig_13.py
    # This takes about 49 minutes
    python3 fig_14.py
    # This takes about 5 minutes
    python3 fig_15.py
    # This takes about 20 minutes
    python3 fig_16.py
    # This takes about 13 minutes
    python3 fig_17a.py
    # This takes about 26 minutes
    python3 fig_17b.py
    # This takes about 27 minutes
    python3 fig_17c.py
    ```

* The json results and PDF figures of each scirpt will be stored inside a new directoty `./results`.

    The results you get may not be exactly the same as the ones shown in the paper due to changes of physical machines.
    And some curves (*e.g.*, cache hit ratio, p99 latency) may fluctuate due to the instability of RNICs in the cluster.
    However, all results here support the conclusions we made in the paper.
