# REU
Walk through of setting up software experiments

# SET UP
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

This is for BQSR/Indel realignment (this step is for the sequencing results and not for the sequencing itsself).
```${HOME}/EVA/scripts/convert_known_snps_indels_to_adam.sh 5```

To see all the options for running experiments you can run the command:
```${HOME}/AVAH/scripts/run_variant_analysis_at_scale.sh -h```



# REFERENCES
https://github.com/MU-Data-Science/EVA
