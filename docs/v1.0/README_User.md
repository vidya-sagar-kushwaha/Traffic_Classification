## Traffic Classification User's Guide

The manual contains all the steps to setup the classification and prioritization modules.
There are two setups in this work.</br>
1. Using 2-interface machine and an access point (AP)</br>
2. Using laptop as AP</br>

### Setup-1:
This setup is shown in the Fig. 1.

It requires</br>
1. One machine with 2 NICs (network interface cards) with Ubuntu 14.04</br>
2. One machine which will act at DHCP server with Ubuntu 14.04</br>
3. One switch </br>
4. One AP</br>
5. User machines (clients)</br>



<div align="center">
<img src="images/setup.png" alt="Fig. 1: 2-interface machine setup" width="650" height="400" />
</div>
<p align="center">Fig. 1: 2-interface machine setup</p>




#### DHCP server ####
1. Install the dhcp server (one time only).<br/>
  ```
   $ sudo apt-get install isc-dhcp-server
  ```   
2. Run the configure-dhcp-server.sh from the folder DHCP.
  ```
  $ cd DHCP
  $ sudo bash configure-dhcp-server.sh
  ```

	It will configure the ip-range  in this machine which will be assigned to any machine which is connected to the switch or AP.

#### 2-interface machine ####
3.  Run the nat.sh and 2interface.sh from 2-interface-machine-files folder.</br>
  ```
	$ cd 2-interface-machine-files

	$ sudo bash nat.sh

	$ sudo 2interface.sh
  ```
4. Run the main-script.sh from DT-classification-2-interface-machine folder.</br>
  ```
  $ cd DT-classification-2-interface-machine

  $ sudo bash main-script.sh
  ```

This is for decision tree based classification. It will classify the traffic into multimedia or download. If the tag is multimedia, then that flow is prioritized immediately.</br>
**Note** : </br>
If you want to use K-NN algorithm approach then you need to run main-script.sh from KNN-classification-2-interface-machine folder.

But before that you need to install weka on 2-interface machine.
  ```
  $ cd
  $ wget https://sourceforge.net/projects/weka/files/weka-3-6/3.6.13/weka-3-6-13.zip/download -O weka-3-6-13.zip
  $ sudo apt-get install unzip
  $ unzip weka-3-6-13.zip
  ```
Now move to the KNN-classification-2-interface-machine directory and run the main script.
  ```
  $ sudo bash main-script-KNN.sh
  ```
This script will show the logs on the screen. It will print the connection ID (4-tuple : source IP, destination IP, source port, destination IP) along with the calculated feature values. It will also show the predicted tag by the classification model.



### Setup 2 : Laptop AP

In this case, we first make a laptop as AP and then perform all the tasks in this laptop itself. See Fig. 2.


<div align="center">
<img src="images/laptop-AP-setup.png" alt="Fig. 2: Laptop AP setup" width="800" height="400" />
</div>
<p align="center">Fig. 2: Laptop AP setup</p>


Here eth0 interface of laptop is connected to wired network (Internet) and wlan0 works as access point. All the classification and prioritization take place at wlan0 interface.



#### Make the laptop as AP
1. Find your laptop wlan0 MAC address
   ```
   $ ifconfig wlan1 | head -1 | rev | cut -d " " -f3 | rev
   ```
Now update the ‘mac-address’ field in the file ‘Laptop-AP’ in Configure-Laptop-as-AP directory.

2. Now run make-ap.sh
   ```
  $ cd Configure-Laptop-as-AP
  $ sudo bash make-ap.sh
   ```

After this command, a wifi network with SSID ‘Laptop-AP’ will be configured. You can connect devices to this network and access multimedia or other content to test the classification model.

After you are done with the experiment, and you want to remove this AP configuration, then run remove-ap.sh


```
  $ sudo bash remove-ap.sh
```
This will remove the changes made to the laptop.

**Classification and Prioritization**</br>
Now, we have set up the laptop as AP. </br>
1. Install python pcapy module (if not already installed)</br>
   ```
  $ sudo apt-get update
  $ sudo apt-get install python-pcapy
   ```
</br>   
2. Go to the ‘DT-classification-on-laptop-AP’ directory and run main-script-laptop-AP.sh. </br>
   ```
  $ cd DT-classification-on-laptop-AP

  $ sudo bash main-script-laptop-AP.sh
   ```
This script will show the logs on the screen. It will print the connection ID (4-tuple : source IP, destination IP, source port, destination IP) along with the calculated feature values. It will also show the predicted tag by the classification model.


### Trace collection and labeling training data ###
This process is common to both the setups.

#### Trace collection ####
As the classification algorithms we have used, require pre-labelled training data, you need to collect traces for both multimedia and download classes.

You can use wireshark or tcpdump tool to capture the traces. Also follow some naming convention for these trace files (pcap files) so that you can identify which pcap file was used to capture multimedia flows and which one was used to capture download flows.

Keep the trace files for both the classes in separate folders.

**For Multimedia**</br>
Move all the multimedia pcap files to Label-training-data/for-multimedia-pcaps directory. Then run create-training-data-for-Multimedia.sh
   ```
   $ bash create-training-data-for-Multimedia.sh
   ```
It will generate a csv file with name as ‘all-multimedia-flows.csv’.
This csv file contains all the multimedia flows along with their feature values.


**For Download**</br>
Move all the download pcap files to Label-training-data/for-download-pcaps directory. Then run create-training-data-for-download.sh</br>
   ```
   $ bash create-training-data-for-download.sh
   ```
It will generate a csv file with name as ‘all-download-flows.csv’.
This csv file contains all the download flows along with their feature values.

Now you need to merge this two csv files so that we have a combined csv file having all the flows. Don’t forget to remove extra header line that you get after merging these two csv files.

Now you can build two models:</br>
1. KNN model</br>
2. Decision tree model</br>

**For KNN model**</br>
1. Move this combined csv file to ‘KNN-classification-2-interface-machine’ directory and rename it to ‘data.csv’</br>
2. Run generate-KNN-model.sh which will generate the knn model.</br>
   ```
   $ bash generate-KNN-model.sh
   ```

**For Decision Tree Model**</br>
1. Move the combined csv file to ‘DT-classification-2-interface-machine’ directory and rename it to ‘data.csv’</br>
2. Run generate-decision-tree-model.sh which will generate the decision tree model.</br>
   ```
   $ bash generate-decision-tree-model.sh
   ```
Above command will generate decision tree rules in hierarchical way, which can be implemented as if-else rules in the classification script (classification-script-decision-tree.py) in the same directory.
