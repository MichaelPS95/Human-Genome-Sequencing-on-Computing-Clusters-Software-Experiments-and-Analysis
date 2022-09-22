# Purpose

The purpose of the software experiments was to gather real world data using experiment configurations based on a Locating Array and to perform analysis on the data. We generated a Locating Array to dictate the experiment settings for each software experiment run, and performed analysis on the four parameter system upon completion of the experiments. The analysis is aimed at finding the individual parameters and two-way interactions between parameters that had the most significant impact on the overall system (in this case the runtime of the software).

# Background

The Locating Array used was generated using code developed by Stephen Seidel (repository linked in references), and shown below.
The 16 and 4 at the top of the file below v2.0 represent the number of configurations and the number of parameters measured respectively. Below that line the 2 3 3 3 specify the number of values each parameter can take on, in this case we are using integers that map to specific settings in the configuration of the cluster computers: Ram, CPU cores, Cluster Configurations, and Network Speed. Below the column of zeroes are the settings for each of the sixteen experiments.
```
v2.0
16	4		
2	3	3	3
0			
0			
0			
0			
0
1	2	2	2
0	1	1	1
0	0	0	0
1	1	0	2
1	2	1	0
1	0	2	1
0	1	2	0
0	2	0	2
1	0	1	1
1	1	1	0
0	0	2	2
0	0	0	1
1	2	0	0
0	1	1	2
1	1	2	1
0	2	0	1
```

# Experiment Setup

Walk through of setting up software experiments assuming you already have a CloudLab account.

You can start an experiment using the profiles I have set up on CloudLab. Please make sure you are using hardware compatible with the software set up process, the list of hardware supported is found [here](https://github.com/MU-Data-Science/EVA/blob/master/Supported_Machines.txt).

**PROFILES**

[Single-Site](https://www.cloudlab.us/show-profile.php?uuid=85de3eb8-e1d9-11ec-aacb-e4434b2381fc)


[Multi-Site](https://www.cloudlab.us/show-profile.php?uuid=100e630b-e1d4-11ec-aacb-e4434b2381fc)

It's important to know that changing the link speed for your nodes is done during the initialization and should be done during this step.
For single-site clusters you can achieve this by going into the view source option when you click on a node set up link above. You should see something in this source file like this:
```
# Creating a link between the nodes.

link = request.LAN("lan")

#Must Provide a bandwidth. BW is in Kbps
link.bandwidth = 1000000
```
If you do not see a field where bandwidth is specified, you can create your own variable and initialize it.

**NOTE:** This is only for the single site profile, as you can go into advanced settings for multi-site and specify link speed via the dropdown menu seen below.
![alt text](https://github.com/MichaelPS95/REU/blob/main/dropdown.png)

Connect to your cluster on vm0 and run the following command to download the code repository:
```
git clone https://github.com/MU-Data-Science/EVA.git
```
To set up the tools used for sequencing, run the following (assuming five node cluster).
```
screen -S setup
cd ${HOME}/EVA/cluster_config
./cluster-configure.sh 5 spark3
```
You can use ctrl+a followed by ctrl+d to detach from the screen while the set up is being done in the screen command. To reattach to that screen, you can use the ```screen -r``` command.


**IMPORTANT:**

Once the ./cluster-configure.sh script is done running, you need to exit the ssh session before moving on to running experiments.

To set up the reference data in the /mydata directory on all the nodes run:
```
screen -S setup
${HOME}/EVA/scripts/copy_files.sh 5
```

While this script is running you can detach and run the following to clone the AVAH repo.
```
cd ; git clone https://github.com/raopr/AVAH.git
cp ${HOME}/AVAH/misc/sampleIDs-vlarge.txt /proj/eva-public-PG0/${USER}-sampleIDs-vlarge.txt
cp ${HOME}/AVAH/misc/sampleURLs-vlarge.txt /proj/eva-public-PG0/${USER}-sampleURLs-vlarge.txt
```

This is for BQSR/Indel realignment (this step is for the sequencing results and not for the sequencing itself).

```${HOME}/EVA/scripts/convert_known_snps_indels_to_adam.sh 5```

To change the allocated cores and RAM for each node, you can follow the steps outlined [here](https://github.com/MU-Data-Science/EVA/blob/master/YARN-README.md).

To see all the options for running experiments you can run the command:

```${HOME}/AVAH/scripts/run_variant_analysis_at_scale.sh -h```

Finally, running an experiment. This is again assuming five node clusters and use of the subgroup comprised of 16 of the 98 sequences from the vlarge files. The REU-IDs.txt and REU-URLs.txt should be present in the directories below assuming the experiment was started with the profiles from above, but both text files are included in this repository if they are not present in the directories for some reason.
 
If sequences need to be downloaded you can run the following commands:
```
hdfs dfs -rm -r /tmp/logs; hdfs dfs -rm -r /spark-events/*; hdfs dfs -rm /*.fastq.gz
${HOME}/AVAH/scripts/run_variant_analysis_at_scale.sh -i /proj/eva-public-PG0/REU-IDs.txt -d /proj/eva-public-PG0/REU-URLs.txt -n 5 -b 2 -p 4 -P H -B
```

If the sequences have already been correctly downloaded you can start and experiment using the following:
```
hdfs dfs -rm -r /tmp/logs; hdfs dfs -rm -r /spark-events/*; hdfs dfs -rm /*.fastq.gz
${HOME}/AVAH/scripts/run_variant_analysis_at_scale.sh -i /proj/eva-public-PG0/REU-IDs.txt -d NONE -n 5 -b 2 -p 4 -P H -B
```

# Experiment Results

You should be able to verify everything went smoothly by checking the log file generated by the experiment. Provided you had to download the sequences you should see somewhere in the log file:
![alt text](https://github.com/MichaelPS95/REU/blob/main/finished%20downloading.png)

To verify that all the sequencing was successful you should see:
![alt text](https://github.com/MichaelPS95/REU/blob/main/finished%20sequencing.png)


Just above the finished sequencing messages for each genome where it says "took #####.##### s", this is the number of seconds the sequencing takes.

# Analysis of Experiment Results

Using software by Yasmeen Akhtar (website listed under references) I was able to feed in the results of the experiments to her program for analysis. The analysis software works by taking in a parameter to specify the number of important factors/interactions you want to search for as well as parameters to specify how to set up the Breadth-First Search model used to do the analysis. The three factors used in the BFS model specify number of fitted models to return, number of children of each node in the tree, and tree height. In the case of this analysis, I chose five important factors with 25 models as well as 25 leaf nodes per node. The results of the analysis suggest that the most impactful parameter to runtime is amount of RAM allocated, followed by cluster configuration, with the interaction between cores and RAM being the third most significant. The drop off in coefficients for other factors and combination of factors suggest those factors and interactions don't carry significant weight in the runtime of the sequencing. The results of the analysis are shown below, and the full analysis with each model can be seen [here](https://github.com/MichaelPS95/Human-Genome-Sequencing-on-Computing-Clusters-Software-Experiments-and-Analysis/blob/main/analysis_1_5_25_25.txt).

```
Occurrence Counts
     R^2 Contr. |           Count |       Magnitude |         AvgMag. | Factor Combination
--------------- | --------------- | --------------- | --------------- | --------------------
        74.9145 |              55 |         13.0932 |        0.238059 | RAM
      0.0743438 |              44 |         3.01354 |       0.0684895 | Cores
        4.15466 |              39 |         2.52132 |       0.0646492 | Cluster_Configuration
              0 |               8 |        0.162505 |       0.0203131 | Network_Speed_Kbps

     R^2 Contr. |           Count |       Magnitude |         AvgMag. | Factor Combination
--------------- | --------------- | --------------- | --------------- | --------------------
        1.68051 |              26 |         2.70024 |        0.103855 | Cores & RAM
      0.0112438 |               8 |        0.141951 |       0.0177439 | Cluster_Configuration & Cores
      0.0107203 |               6 |        0.128542 |       0.0214237 | Network_Speed_Kbps & Cores
     0.00658929 |               4 |       0.0635728 |       0.0158932 | Cluster_Configuration & RAM
     0.00306273 |               2 |        0.033962 |        0.016981 | Cluster_Configuration & Network_Speed_Kbps
```

# REFERENCES

https://github.com/MU-Data-Science/EVA

https://github.com/sseidel16/v4-la-tools

https://sites.google.com/view/akhtar2019/home?authuser=0
