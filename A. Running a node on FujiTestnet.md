
## Run node on aws: https://docs.avax.network/nodes/build/setting-up-an-avalanche-node-with-amazon-web-services-aws
### Some things to take care of :
1. Choose a storage optimized instance in the beginning i.e till you node will download all the data blobs and write to disc.
    Dont choose Arm processor.
    i3.large	
    CPU: Equivalent of 2 AWS vCPU
    RAM: 15.25 GiB
    Storage: 500 GB (choose gp3 type)
    OS: Ubuntu 18.04/20.04 or MacOS >= Catalina
    Will probably take a day.

2. After it has downloaded all the data, Stop your instance and change your instance type to memory optimized instance on AWS.
    Dont choose Arm processor.
    r6i.large	
    CPU: Equivalent of 2 AWS vCPU
    RAM: 16 GiB
    Storage: 500 GB (choose gp3 type)
    OS: Ubuntu 18.04/20.04 or MacOS >= Catalina
    Will probably take a day.

3. Dont forget to attach Elasticip with your server.
3. At the time of downloading, The disc space used by Fuji testnet is around 117GB .

4. Download and start avalanchego to start syncing blockchain data on your server.
    ```
    wget -nd -m https://raw.githubusercontent.com/ava-labs/avalanche-docs/master/scripts/avalanchego-installer.sh;\
    chmod 755 avalanchego-installer.sh;\
    ./avalanchego-installer.sh
    ```


### Reusing DB data on other avalanche nodes.
The above mentioned steps download all the blockchain data on your root volume. If you want to start other validator node 
or would like to restore your unusable alidator node then you will have to download the whole blockchain data again. Which is kind 
of a really slow process. We can solve it by using EBS volumes on amazon web services. 

#### Attach an EBS volume to your server, 500GB is sufficient. Follow these steps:
1. Open the Amazon EC2 console at https://console.aws.amazon.com/ec2/.
2. In the navigation pane, choose Volumes.
3. Select the volume to attach and choose Actions, Attach volume or click on create volume (Choose the zone in which your server is present)
    1. choose volume type gp3
    2. 500 GB volume size.
    3. Increase iops to 5000.
    4. Increase throughput to 300 MiB/s.


4.  For Instance, enter the ID of the instance or select the instance from the list of options.
    The volume must be attached to an instance in the same Availability Zone.
5. For Device name, enter a supported device name for the volume. This device name is used by Amazon EC2. 
    The block device driver for the instance might assign a different device name when mounting the volume. 
    For more information, see **[Device names on Linux instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/device_naming.html)** .
6. Choose Attach volume.

#### Reuse Blockchain Data to create other Validator nodes using EBS volume.
1. Loginto your server with the ssh key.
2. Use the lsblk -f command to get information about all of the devices attached to the instance.
    ```
        sudo lsblk -f
    
        NAME		FSTYPE	LABEL	UUID						MOUNTPOINT
        nvme1n1	        xfs		7f939f28-6dcc-4315-8c42-6806080b94dd
        nvme0n1
            ├─nvme0n1p1	xfs	    /	90e29211-2de8-4967-b0fb-16f51a6e464c	        /
            └─nvme0n1p128
        nvme2n1
    ```
    For example, the following output shows that there are three devices attached to the instances—nvme1n1, nvme0n1, and nvme2n1. 
    The first column lists the devices and their partitions. The FSTYPE column shows the file system type for each device. 
    If the column is empty for a specific device, it means that the device does not have a file system. In this case, device nvme1n1 and 
    partition nvme0n1p1 on device nvme0n1 are both formatted using the XFS file system, while device nvme2n1 and partition nvme0n1p128 
    on device nvme0n1 do not have file systems.
    If the output from these commands show that there is no file system on the device, you must create one.



2.  If you discovered that there is a file system on the device in the previous step, skip this step. If you have an empty volume, 
    use the mkfs -t command to create a file system on the volume.
    ```
    sudo mkfs -t ext4 /dev/xvdf
    ```
    If you get an error that mkfs.xfs is not found, use the following command to install the XFS tools and then repeat the previous command:
    [user ~]$ sudo apt-get install xfsprogs

3.  Use the mkdir command to create a mount point directory for the volume. The mount point is where the volume is located in the file system 
    tree and where you read and write files to after you mount the volume. Lets create a directory named /blockchaindata.
    ```
        [user ~]$ sudo mkdir /blockchaindata
    ```
    Use the following command to mount the volume at the directory you created in the previous step.
    ```
    [user ~]$ sudo mount /dev/xvdf /blockchaindata
    ```

4. Now start this command 
    ```
    ./avalanchego-installer.sh  --db-dir /blockchaindata --fuji
    ```
    All your blockchain data will be stored on EBS volume mouunted on  /blockchaindata. Wait for it to sync.
    Let it sync till this command on your server returns True
    ```
    curl -X POST --data '{
        "jsonrpc":"2.0",
        "id"     :1,
        "method" :"info.isBootstrapped",
        "params": {
            "chain":"X"
        }
    }' -H 'content-type:application/json;' 127.0.0.1:9650/ext/info
    ``` 


5. Once your blockchain data is synced. Stop avalanchego service by using this command.
    ```
    sudo systemctl stop avalanchego
    ```
6. Take snapshot of your EBS volume by following these steps:

    1. Open the Amazon EC2 console at https://console.aws.amazon.com/ec2/.
    2. In the navigation pane, choose Snapshots, Create snapshot.
    3. For Resource type, choose Volume.
    4. For Volume ID, select the volume from which to create the snapshot.
        (Optional) For Description, enter a brief description for the snapshot.
        (Optional) To assign custom tags to the snapshot, in the Tags section, choose Add tag, and then enter the key-value pair. You can add up to 50 tags.
    5. Choose Create snapshot.
    6. Since data is huge, The snapshot will time to complete.

7. Now, Create a new EBS volume from the snapshot you have created at step 6 above. 
8. Create a new server and attach EBS volume created at step 7.
9. Loginto your server and mount this EBS volume without reformating it since it will have all the blockchain data in it.
10. Start avalanchego service and it will start syncing without downloading all the blockchain data.


