# **Introduction** #

SCFS is a cloud-backed file system that provides strong consistency even on top of eventually-consistent cloud storage services. Its build on top of [FUSE](http://fuse.sourceforge.net/), thus providing a POSIX-like interface. SCFS provides also a pluggable backend that allows it to work with a single cloud or with a cloud-of-clouds. The design, implementation and evaluation of the system is described in our [USENIX ATC'14 paper](http://www.di.fc.ul.pt/~bessani/publications/usenix14-scfs.pdf).

It main concern is address some of the problems presented on the other existing cloud-backed file systems:

  1. Most of them not support controlled file sharing among clients.
  1. Some, use a proxy to connect with the cloud storage, which is a single point of failure, once if it is down, no client can access to the data stored at the cloud side.
  1. Almost all file systems use only one single cloud as backend.

You can download SCFS for Linux distributions in [Downloads](Downloads.md). To get some assistance in SCFS installation and configuration, please take a look at [HowToUseSCFS](HowToUseSCFS.md).

# **Architecture** #

The figure bellow presents the architecture of SCFS with its three main components: the backend cloud storage for maintaining the file data; the coordination service for managing the metadata and to support synchronization; and the SCFS Agent that implements most of the SCFS functionality, and corresponds to the file system client mounted at the user machine.

![http://depsky.googlecode.com/svn/images/scfs-3.png](http://depsky.googlecode.com/svn/images/scfs-3.png)

The figure shows the backend cloud storage as cloud-of-clouds formed by Amazon S3, Google Storage, RackSpace Cloud Files and Windows Azure. To store data in this clouds, SCFS uses DepSky. More specifically, uses DepSky-CA protocol once we want address the main cloud storage limitations (see DepSky). SCFS allows also a different backend formed by only one cloud.

The coordination services used by SCFS are [DepSpace](http://www.navigators.di.fc.ul.pt/software/depspace/) and ZooKepper. In the figure above they are installed in computing clouds, but they can be installed on any IaaS or in any server. To get more information about theses system please read the referred papers. [DepSpace](http://www.navigators.di.fc.ul.pt/software/depspace/) is described on a [EuroSys'08 paper](http://www.di.fc.ul.pt/~bessani/publications/eurosys08-depspace.pdf) and ZooKepper on a [Usenix'10 paper](https://www.usenix.org/legacy/event/usenix10/tech/full_papers/Hunt.pdf).

The SCFS Agent component is the one that implements the most features of the file system. It makes a lot of usage of cache techniques to improve the system performance, to reduce the monetary costs and to increase the data availability. More Specifically it uses a temporary small memory cache to absorb some metadata update bursts, a main memory cache to maintain data of open files, and a bigger disk cache to save all the most recents files used by the client. The last one uses all the free space in disk. After the disk is full it uses LRU policies to create new space.

# **Operation Modes** #

SCFS is a very configurable file system, once clients can use it according with their needs. There is four main configurations that change both the file system behaviour as the provided guarantees:

  1. The system can be configured to use only one cloud or a cloud-of-clouds for both the backend cloud storage as to the coordination service.
  1. The client can choose if it uses the main memory cache to store open files data, or not.
  1. The sending of data to the clouds can be synchronous or asynchronous.
  1. The system can be _sharing_ or _non-sharing_. If _non-sharing_ configuration is used the system does not uses none coordination service. Is considered that all files are private (not shared), therefore all the metadata can be stored in the storage clouds.