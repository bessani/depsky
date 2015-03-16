A programming library that implements the DepSky cloud-of-clouds replication algorithms. These algorithms use Byzantine quorum systems together secret sharing and erasure codes to spread data in a diverse set of clouds ensuring provider fault tolerance and confidentiality.

DepSky was initially described on a [EuroSys'11 paper](http://www.di.fc.ul.pt/~bessani/publications/eurosys11-depsky.pdf) and later on fully described and analyzed in an [ACM Transactions on Storage paper](http://www.di.fc.ul.pt/~bessani/publications/tos13-depsky.pdf).

This implementation supports the read and write protocols with replication and confidentiality. The system is organized around three main components:

  * DepSky Byzantine quorum protocols, just as described in the papers
  * [JSS](JSS.md): A Java implementation of the PVSS secret sharing algorithm
  * [JEC](JEC.md): A simple Java implementation of the Reed-Solomon erasure code

Currently DepSky supports the following services/providers:

  * Amazon S3
  * Google Storage
  * Rackspace Files
  * Windows Azure Blob Storage

To understand how to use DepSky read the wiki page HowToUseDepSky.

We provide also a file system that uses DepSky as backend. It is called SCFS. You can find a short description of SCFS [here](https://code.google.com/p/depsky/wiki/SCFS), a download zip [here](https://code.google.com/p/depsky/wiki/Downloads), and a 'how to use' wiki page [here](https://code.google.com/p/depsky/wiki/HowToUseSCFS).

DepSky is funded by the European Commission through the TClouds FP7 project.