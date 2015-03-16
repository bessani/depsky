# **Getting Started with SCFS** #

This page describe how you can configure, install and use the available version of SCFS. The version we provide uses DepSpace as coordination service and Amazon S3 as single cloud storage backend. Take a look at the [SCFS](SCFS.md) wiki page before continuing reading this page.

### SCFS ###
The first step is go to the [Downloads](Downloads.md) page and get the latest version available of SCFS. After the download is finished, you need to extract the archive downloaded. Make sure you have java 1.7 or later installed.

Next, you need to link the system with the Fuse on your machine. To do that, you just need to run a script we provide in the project's root directory. If you have a x64 Java installed, you just need to run:

```
./configure_x64.sh
```

In the other hand, if your java is a x86 version, please run:

```
./configure_x86.sh
```

In the next step you just need to set your JAVA\_HOME and FUSE\_HOME in the build.conf file. Usually, your fuse home will be in /usr/local. The JAVA\_HOME will depend on the java version you have installed on your machine.

The next step is to configure the remain configuration files. More specifically, it is necessary to fill configuration files for DepSky and [DepSpace](http://www.navigators.di.fc.ul.pt/software/depspace/). The following subsections explain how to do it.

### DepSky ###

To fill DepSky configuration files please read the **'Getting Started with DepSky'** section in HowToUseDepSky page. Here you can learn how to create the clouds storage accounts and where can you get the necessary API keys.

If you want to share files with other users, you have also to fill the _canonicalId_ field in each entry of the accounts.properties. The version avilable of the system only allows sharing files between different accounts if you use only Amazon S3 (both in one single cloud  and cloud-of-clouds backends). You can find your canonical Id in the Security Credentials page of your AWS Management Console.

You can use also SCFS with only one cloud storage as backend. For now, we only support Amazon S3. To configure it you just need to delete the last three entries in the account.properties file (see HowToUseDepSky). Your file must be equal to the content below:

```
#AMAZON
driver.type=AMAZON-S3
driver.id=cloud1
accessKey=********************
secretKey=****************************************
location=EU_Ireland
canonicalId=******************************************************
```

### DepSpace ###

[DepSpace](http://www.navigators.di.fc.ul.pt/software/depspace/) configuration consists in two main steps; the address and port configuration of all DepSpace's replicas, and the configuration of system parameters. This could be done in config/hosts.config and config/system.config files respectively. To comment out a configuration parameter in those files, the line must start with the character #.

DepSpace is built on top of [BFT-SMaRt](http://code.google.com/p/bft-smart/). Please take a look at [How to configure BFT-SMaRt replicas](http://code.google.com/p/bft-smart/wiki/Configuration).

In hosts.config file you can set the address and port of all DepSpace replicas. The configuration file should look like this:

```
#server id, address and port (the ids from 0 to n-1 are the service replicas) 
0 127.0.0.1 11000
1 127.0.0.1 11010
2 127.0.0.1 11020
3 127.0.0.1 11030
4 127.0.0.1 11040
5 127.0.0.1 11050
6 127.0.0.1 11060
7 127.0.0.1 11070
```


After the configuration of the replicas address and port, you can set the parameter Besides the parameters you need to set to [BFT-SMaRt](http://code.google.com/p/bft-smart/), there are others you need to configure to [DepSpace](http://www.navigators.di.fc.ul.pt/software/depspace/). These parameters are described below:

| **Parameter** | **Description** | **Values** | **Default value** |
|:--------------|:----------------|:-----------|:------------------|
| system.general.rdRetryTime | The time the system blocks on rd operation waiting for a match tuple in the space. SCFS does not uses this operation. | Integer (miliseconds) | 10000 |
| system.general.realTimeRenew | Use or not of renewable tuples. Note that SCFS needs this parameter to be true. | true or false | true |
| system.general.logging | Use or not of logging. Set this parameter to true if you want it. | true or false | false |
| system.general.logging.mode | Set the logging mode. Set this parameter to 0 if you have set the logging use to false. The other modes are: 1="rw", 2="rws", (any other)="rwd" | Integer | 0 |
| system.general.batching | Use batches of messages. We recommend you to set this to false. | true or false | false |
| system.general.checkpoint\_period | The period in which the system performs a checkpoint of its state. | Integer (number of operations) | 1000 |
| system.general.replaceTrigger | The usage or not of the DepSpace's replace trigger layer. Note that SCFS needs this layer to correctly work. | true or false | true |

We recommend you to use the default values set in the config/system.config file in the SCFS.zip file available in [Downloads](Downloads.md).

#### Public Keys ####
We provide some default DepSpace keys. However, if you need to generate new public/private keys for DepSpace replicas or clients, you can use the following command to generate files for public and private keys:

./rsaKeyPairGenerator.sh <id of the replica/client>

Keys are stored in the config/keys folder. The command above creates key pairs both for clients and replicas. Currently such keys need to be manually distributed.

### **Using DepSpace with just 1 replica** ###

The default system.config and hosts.config file are set to use Byzantine Fault tolerant DepSpace (using 3f+1 replicas).

To use DepSpace without any fault tolerance guarantee, some modification need to be done in the config/system.config file. In this way you need to set the following parameters of system.config file:

```
#Number of servers in the group
system.servers.num = 1

#Maximum number of faulty replicas 
system.servers.f = 0

#Initial View 
system.initial.view = 0
```

You also need to reconfigure the replicas address/port file. So, the config/hosts.config file should look like the following:

```
#server id, address and port (the ids from 0 to n-1 are the service replicas) 
0 127.0.0.1 11000
```

# **Deploying DepSpace** #
To deploy DepSpace in a server (being it in the clouds or in other location), you first need to get the files to deploy. To generate the DepSpace files to deploy you just need to run the command below:

```
./generateDepSpaceToDeploy.sh
```

This script will generate a zip file (DepSpace.zip) with the configurations you did for the DepSpace locally (namely the hosts.config file). To deploy DepSpace in some sites, you just need to upload the DepSpace.zip these sites and unzip it:

```
unzip DepSpace.zip
```

After unzip the zip file, to run the DepSpace in each site, you just need to run the runDepSpace script in each site. You can find this script inside the folder DepSpace after you unzip the file. To execute this script you should run the follow command:

```
./runDepSpace.sh <replicaID>
```


# **Mounting SCFS** #

To mount SCFS you just need to run the following command in the root of the project:

```
./mountSCFS.sh <mountPoint> <clientID>
```

The meaning of the two given arguments are:

  * _mountPoint_ - the folder where SCFS will be mounted;

  * _clientID_ - the identifier of the client (must be unique).

Each time you mount the system you are able to define how the system will behave, opting for several parameters. This parameters will have influence in the system guarantees and performance. Note that the default behavior of the system (the command presented before) provides the highest level of guarantees and the lower performance.

In the table bellow are shown all the flags that can be turned on:


| **Parameter** | **Description** | Default value |
|:--------------|:----------------|:--------------|
| -optimizedCache | Use of main memory data cache. | off |
| --sync-to-cloud | The FSYNC POSIX operation flushes the data to the clouds. If turned off, FSYNC flushes data to local disk. | off |
| --non-blocking-cloud | Data are sent to the clouds asynchronously. If turned off, the file close operation will block when data are sent to the clouds. | off |
| --non-sharing | The system runs in non-sharing mode of operation. It means that clients are not able to share data between them | off |
| -printer | Turn on the debug mode. | off |

As you can understand, if the _--non-blocking-cloud_ flag is turned on, the performance of the system is increased while the guarantees offered by the close operation decrease.

## **Non-Sharing SCFS** ##

To use this operation mode, you should turn on the _--non-sharing_ flag as shown bellow:

```
./mountSCFS.sh <mountPoint> <clientID> --non-sharing
```

This operation mode is ideal to use the system with its highest performance, backuping the file system data to the clouds. So, the use of _--non-blocking-cloud_ flag are recommended:

```
./mountSCFS.sh <mountPoint> <clientID> --non-sharing --non-blocking-cloud
```

**Note:** Since all files are personal in this operation mode, it does not need the DepSpace deployment. So, if you want to try SCFS without experience the deployment effort of DepSpace, this is the perfect option to do that.

# **Using SCFS** #

To use SCFS you just need to enter in the `<mountPoint>` folder and operate in the same way you do in your local filesystem. For example, let your SCFS configuration be:

```
./mountSCFS.sh /scfs 4 --non-sharing --non-blocking-cloud -optimizedCache
```

In the example, your `<mountPoint>` will be _/scfs_. So, you just need to operate over the files in that folder to take advantage of SCFS.

### **Share a file using SCFS** ###

To share a file, you just need to use the _setfacl_ command, giving the clientId of the client with which you want to share a file, and the permissions to that client:

```
setfacl -m u:<friendClientID>:<permissions> <fileName>
```

For example, assuming there is a file named _file1.txt_, to share this file with the client 5 giving full control over the file, you just need to run the following command:

```
setfacl -m u:5:rwx file1.txt
```