# SMART: A High-Performance Adaptive Radix Tree for Disaggregated Memory

This is the implementation repository of our *OSDI'23* paper: **SMART: A High-Performance Adaptive Radix Tree for Disaggregated Memory**.
This artifact provides the source code of **SMART** and scripts to reproduce all the experiment results in our paper.
**SMART**, a di<u>**S**</u>aggregated-me<u>**M**</u>ory-friendly <u>**A**</u>daptive <u>**R**</u>adix <u>**T**</u>ree, is the first radix tree for disaggregated memory with high performance.

This README is specifically for **artifact evaluation (AE)**.

- [SMART](#smart-a-high-performance-adaptive-radix-tree-for-disaggregated-memory)
  * [Supported Platform](#supported-platform)
  * [Create Cluster](#create-cluster)
  * [Source Code of SMART *(Artifacts Available)*](#source-code-of-smart-artifacts-available)
  * [Environment Setup](#environment-setup)
  * [YCSB Workloads](#ycsb-workloads)
  * [Getting Started *(Artifacts Functional)*](#getting-started-artifacts-functional)
  * [Reproduce All Experiment Results *(Results Reproduced)*](#reproduce-all-experiment-results-results-reproduced)

## Supported Platform
We strongly recommend you to run SMART using the r650 instances on [CloudLab](https://www.cloudlab.us/) as the code has been thoroughly tested there.
We haven't done any test in other hardware environment.

We have reserved 16 r650 nodes on CloudLab **from May 2nd to May 25th** for artifact evaluation.

* You can simply use a ***temporary account*** to use our reserved r650 nodes on CloudLab.
Contact us and we will provide you with the temporary account.

* If you want to conduct AE with your own account, contact us and we will release the reserved nodes.


## Create Cluster
Follow the following steps to create our experimental cluster on CloudLab:

1) Log into the temporary account or your own account. If you have trouble logging into the temporary account, please contact us for assistance.

2) Now you have logged into Cloublab console.
If you are using our temporary account, please submit the SSH public key of your personal computer via `[Account Name]`|-->`Manage SSH keys`, after which you can go on building the experiment environment since May 2nd.

    Otherwise, if you are using your own account and there are not 16 r650 machines available, please submit a reservation **in advance** via `Experiments`|-->`Reserve Nodes`.  If you have trouble reserving nodes, please contact us for assistance.

3) Click `Experiments`|-->`Create Experiment Profile`.
Upload `./script/cloudlab.profile` provided in this repo.
Input a file name (*e.g.*, SMART) and click `Create` to generate the experiment profile for SMART.

4) Click `Instantiate` to create a 16-node cluster using the profile (This takes about 7 minutes).

5) Try logging into and check each node using the SSH commands provided in the `List View` on CloudLab. If you find some nodes have broken shells (which happens sometimes in CloudLab), you can reload them (Click `Reload Selected` in the `List View`).


## Source Code of SMART *(Artifacts Available)*
Now you can log into all the 16 r650 Cloudlab nodes. Using the following command to clone this github repo in the home directory of **all** nodes:
```shell
git clone https://github.com/dmemsys/SMART.git
```


## Environment Setup
You have to install the necessary dependencies in order to build SMART.
Note that you should run the following steps on **all** nodes you have created.

1) Set bash as the default shell. And enter the SMART directory.
    ```shell
    sudo su
    chsh -s /bin/bash
    cd SMART
    ```

2) Install Mellanox OFED.
    ```shell
    # It doesn't matter to see "Failed to update Firmware"
    # This takes about 8 minutes
    sh ./script/installMLNX.sh
    ```

3) Resize disk partition.

    Since the r650 nodes remain a large unallocated disk partition by default, you should resize the disk partition using the following command:
    ```shell
    # It doesn't matter to see "Failed to remove partition" or "Failed to update system information"
    sh ./script/resizePartition.sh
    # This takes about 6 minutes
    reboot
    # After rebooting, log into all nodes again and execute:
    sudo su
    resize2fs /dev/sda1
    ```

4) Enter the SMART directory. Install libraries and tools.
    ```shell
    cd SMART
    # This takes about 3 minutes
    sh ./script/installLibs.sh
    ```

5) HugePages setting.
    ```shell
    echo 36864 > /proc/sys/vm/nr_hugepages
    ulimit -l unlimited
    ```


## YCSB Workloads

You should run the following steps on **all** nodes.

1) Download YCSB source code.
    ```shell
    cd ./ycsb
    curl -O --location https://github.com/brianfrankcooper/YCSB/releases/download/0.11.0/ycsb-0.11.0.tar.gz
    tar xfvz ycsb-0.11.0.tar.gz
    mv ycsb-0.11.0 YCSB
    ```
2) Download the email dataset for string workloads.
    ```shell
    gdown --id 1ZJcQOuFI7IpAG6ZBgXwhjEeKO1T7Alzp
    ```

3) We first generate a small set of YCSB workloads here for **quick start** (*i.e.*, kick-the-tires).
    ```shell
    # This takes about 2 minutes
    sh generate_small_workloads.sh
    ```


## Getting Started *(Artifacts Functional)*

* Return to the SMART root directory and execute the following commands on **all** nodes to compile SMART:
    ```shell
    mkdir build; cd build; cmake ..; make -j
    ```

* Execute the following command on **one** node to initialize the memcached:
    ```shell
    /bin/bash ../script/restartMemc.sh
    ```

* Execute the following command on **all** nodes to split the workloads:
    ```shell
    python3 ../ycsb/split_workload.py <workload_name> <key_type> <CN_num> <client_num_per_CN>
    ```
    * workload_name: the name of the workload to test (*e.g.*, `a` / `b` / `c` / `d` / `la`).
    * key_type: the type of key to test (*i.e.*, `randint` / `email`).
    * CN_num: the number of CNs.
    * client_num_per_CN: the number of clients in each CN.

    **Example** (kick-the-tires):
    ```shell
    python3 ../ycsb/split_workload.py a randint 16 24
    ```

* Execute the following command in **all** nodes to conduct a YCSB evaluation:
    ```shell
    ./ycsb_test <CN_num> <client_num_per_CN> <coro_num_per_client> <key_type> <workload_name>
    ```
    * coro_num_per_client: the number of coroutine in each client (2 is recommended).

    **Example** (kick-the-tires):
    ```shell
    ./ycsb_test 16 24 2 randint a
    ```

* Results:
    * Throughput: the throughput of **SMART** among all the cluster will be shown in the terminal of the first node (with 10 epoches by default).
    * Latency: execute the following command in **one** node to calculate the latency results of the whole cluster:
        ```shell
        python3 ../us_lat/cluster_latency.py <CN_num> <epoch_start> <epoch_num>
        ```

        **Example** (kick-the-tires):
        ```shell
        python3 ../us_lat/cluster_latency.py 16 1 10
        ```

## Reproduce All Experiment Results *(Results Reproduced)*
We provide code and scripts in `./exp` folder for reproducing our experiments. For more details, see [./exp/README.md](./exp).

