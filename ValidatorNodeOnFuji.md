
#### Run an Avalanche Node with Amazon Web Services (AWS)

Follow these steps to spawn a validator node for FUJI testnet on AWS. https://docs.avax.network/nodes/build/setting-up-an-avalanche-node-with-amazon-web-services-aws

Some notes:
1. Choose compute optimized nodes on AWS with Ubuntu 22.04 LTS (options available here: https://aws.amazon.com/ec2/instance-types/)
2. If you want to limit connections to your nodes from specific ips, add them in the security group with port 9650.
3. For hard disks, choose gp3 devices with at least 3000 IOPS and 125 Throughput. Whats the difference you say
    I/O is the number of accesses to the disk. Each time you need to read a file, you need "at least" to access once to the file. However the content is read in "chunks", 
    each time you read a "chunk" a new I/O is requested. Imagine bitting a Chocolate bar, you need at least to access once to the chocolate bar, and you start bitting (I/O) 
    until you end it. Each bite is a I/O. You need several I/Os to swallow the whole bar.

    IOPS is I/O per second. Speed. So basically how fast we can perform each bite in the chocolate bar. 
    A IOPS EBS is a volume specialized in performing fast biting: Ñam-Ñam-Ñam vs Ñam------Ñam-----Ñam

    Throughput is the amount of info you read in each I/O. Following with the example, you can eat the whole Chocolate bar in two different ways, 
    small bites (small throughput) or big bites(big throughput) it will depend on your mouth size. Througput EBS volume is specialized in performing big biting: Ñam vs Ñaaaaaaaam

    Is I/O and Throughput related? Sure. If you have to read a big file from EBS, and your througput is small (aka, your mouth is small so your bites are small) 
    then you need to access (I/O) more frequently until the file is completely read. Ñam-Ñam-Ñam-Ñam
    On the other hand, if you have a big mouth (big throughput), then you will need less bites and less I/O. Ñaaaam---Ñaaaam

    Every block downloaded will be written to disc and every RPC request served will have to read from the disc.
    In our case, Increasing IOPS will be more beneficial as compared  throughput.

4. If your server is syncing too slow, Try to increase IOPS.
5. If at sometime, you are falling short of disk size. Resize it using AWS console. https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/requesting-ebs-volume-modifications.html
6. I have uploaded the whole dataset of FujiTest on s3. Feel free to contect me for a download.