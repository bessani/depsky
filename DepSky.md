# **Introduction** #

DepSky is a system that improves the availability, confidentiality and integrity of stored data in the cloud. It reaches this goal by encrypting, enconding and replicating all the data on a set of differents clouds, forming a cloud-of-clouds. For the current implementation of the system and for the text below we consider a cloud-of-clouds formed by four clouds.

More specifically DepSky address four important limitations:

  * **Loss of availability** - DepSky addresses this limitation because it replicates all the data in a set of clouds, and even if some of them presents some problems, all the data will be available if a subset of them are reachable.

  * **Loss an corruption of data** - DepSky deals with this problem using Byzantine fault-tolerance replication to store data in a cloud-of-clouds, being possible get the data correctly even if some of the clouds corrupt or lose data.

  * **Loss of privacy** - DepSky employs a secret sharing scheme and erasure codes to ensure that all data that will be stored in a cloud-of-clouds is in ciphertext.

  * **Vendor lock-in** - DepSky addresses this limitation because unlike use a single cloud provider, use a set of them.

# **Protocols** #

Below is a brief explanation of the DepSky protocols to store data in a cloud-of-clouds. All of them replicate the data for all clouds used but only is ensured that the data is properly stored in three (due to the Byzantines quoruns).

### **DepSky-A** ###

This protocol replicates all the data in clear text in each cloud.

### **DepSky-CA** ###

This protocol uses secret sharing and erasure code techniques to replicate the data in a cloud-of-clouds. The image below show how this is donne. First is generated an encryption key, and after that the original data block is encrypted. Then the encrypted data block is erasure coded and are computed key shares of the encryption key. In this case we get four erasure coded blocks and four key shares because we use four clouds. Lastly, is stored in each cloud a different coded block together with a different key share.

![http://depsky.googlecode.com/svn/images/codingdata.png](http://depsky.googlecode.com/svn/images/codingdata.png)

### **DepSky-only-JSS** ###

This protocol only use secret sharing. Basically, is generated an encryption key and the data is encrypted. Then is generated four key shares of the key. Finally are spread by each cloud the data encrypted together with a different key share.

### **DepSky-only-JEC** ###

On the other hand, this protocol only use erasure codes to replicate the data. The data is erasure coded in four different blocks and then each of them is stored in a different provider.

This protocol may be useful to those who your application already encrypt the data.

# **Costs** #

As would be expected, a DepSky client would be required to pay four (using a cloud-of-clouds of four cloud providers) times more than he would pay if uses a single cloud. That not happens (if using DepSky-CA protocol) due to the erasure codes techniques. The erasure codes technique used (see [JEC](JEC.md)) allow us to store in each of the four cloud providers only half of the orginal block data size. So, using DepSky, the client only will pay twice more than using a single cloud.


For more information see the DepSky paper. You can find it here [EuroSys'11 paper](http://www.di.fc.ul.pt/~bessani/publications/eurosys11-depsky.pdf).