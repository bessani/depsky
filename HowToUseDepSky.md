# **Getting Started with DepSky** #

This section explains you how to create the providers accounts to form a cloud-of-clouds environment. If you want to test DepSky without create the accounts, you can use local storage instead. Please read the next section called **Testing DepSky**.

First of all, go to the [Downloads](Downloads.md) page and get the latest stable version available. After the download is finished, you need to extract the archive downloaded. Make sure you have java 1.7 or later installed.

Done this, you need to fill up the accounts.properties file (you can find it inside the config folder). To fill up this file you need first create accounts in the cloud providers we support. To do that follow the links below:

  * Amazon S3 - http://aws.amazon.com/s3/
  * Google Storage - https://developers.google.com/storage/
  * RackSpace - http://www.rackspace.co.uk/
  * Windows Azure - https://www.windowsazure.com/en-us/

After create the accounts you have access to yours API keys and so, you can fill up the accounts.properties file. To help you to find your keys, follow the steps below.

  * To find Amazon s3 keys go to the AWS Management Console, click in S3 service, now, in the upper right corner click in your account name and go to the Security Credentials. After that, in the Access Keys separator you can generate your access and secret keys.

  * To find Google Storage keys go to the Google API Console, and then go to the Google Cloud Storage separator. Now choose Interoperable Access and there you can find your keys. Don't forget first enable Google Cloud Storage in the services separator.

  * To find RackSpace keys, go to the Control Panel. In the upper right corner you can find how to get your secret key. The access key is just your login username.

  * To find Windows Azure keys go to the windows azure portal. First you need to create a new storage project. After select this new project, at the bottom of the page, you can find the key management. In this case your access key is your storage project name and you secret key is the primary key in the key management.

If you only want to use Amazon S3 as your cloud storage provider, you can only create one account at Amazon S3 and use the example file provided (config/accounts\_amazon.properties). To do that, copy the content of the 'accounts\_amazon.properties' file to the one mentioned before (config/accounts.properties). In this case will be used four different Amazon S3 locations to store the data (US\_Standard, EU\_Ireland, US\_West and AP\_Tokyo).

Now all the setup is finished and DepSky is ready to be used.



# **Testing DepSky** #

To test DepSky we provide a simple main that can be found in `src.depskys.core.LocalDepSkySClient`. To run this main use the DepSky\_Run.sh scritp at the root of the project providing 3 arguments:

  * The first one is the client id (for now use ids below 6 because we only have keys generated for ids until the 6).
  * The second argument indicates what protocol will be used to replicate the data. There are 4 possibilities:

> -> 0 means that will be used DepSky-A (there is no erasure codes neither secret sharing).

> -> 1 to use DepSky-CA (use erasure codes and secret sharing).

> -> 2 for use only erasure codes.

> -> 3 to use only secret sharing.

  * The third argument indicated the storage location.

> -> 0 if you want to use cloud storage to replicate the data.

> -> 1 if you want to store all the data locally (testing purposes). If you want to use the local storage you need first run the server that can be found in `src.depskys.clouds.drivers.localStorageService.ServerThread`. To run this server you can use the Run\_LocalStorage.sh script at the root fo the project. This server will receive all requests at ip 127.0.0.1 and port 5555.

Let´s do an example to exemplify. If we run DepSky with the command below, we gonna start a session with the client id 0, all the data will be replicated using erasure codes and secret sharing and will be stored on the cloud providers.

```
$ ./DepSky_Run 0 1 0
```

This main allow you to read, write and delete. You have five commands available:

  * `pick_du 'name'` - will change the container that you are using to read and write.
  * `write 'data'` - will write a new version with the content 'data' over the container selected.
  * `read` - will read the last version written to the container selected
  * `delete` - will delete all the data (data and metadata files) associated with the container selected.
  * `read_m 'num'` - will read old versions over the container selected. If 'num' = 0, will read the last version written, if 'num' = 1, will read the penultimate version written, etc. Note that only is possible read old versions written in this session because this main maintain all the information in memory. To read all the old versions this main must be changed.

This main is not enough to take advantage of all the functionalities provided by DepSky. To learn more about all you can do with DepSky read the section below.

# **Using DepSky as a Library** #

To start, you need to create a `src.depskys.core.LocalDepSkySClient` object. As you can see below, the constructor receive the client id and a boolean. If the boolean value is set to false, will be used the **local storage**, otherwise will be used the **cloud storage**.

```
  public LocalDepSkySClient(int clientId, boolean useModel) throws StorageCloudException {

  	this.clientId = clientId;
  	DepSkySKeyLoader keyLoader = new DepSkySKeyLoader(null);
  	if(!useModel){
  		this.cloud1 = new LocalDiskDriver("cloud1");
		this.cloud2 = new LocalDiskDriver("cloud2");
		this.cloud3 = new LocalDiskDriver("cloud3");
		this.cloud4 = new LocalDiskDriver("cloud4");
		this.drivers = new IDepSkySDriver[]{cloud1, cloud2, cloud3, cloud4};
	}else{	
		List<String[][]> credentials = null;
		try {
			credentials = readCredentials();
		} catch (FileNotFoundException e) {	
			System.out.println("accounts.properties file dosen't exist!");
			e.printStackTrace();
		} catch (ParseException e) {
			System.out.println("accounts.properties misconfigured!");		
			e.printStackTrace();
		}
		this.drivers = new IDepSkySDriver[4];
		String type = null, driverId = null, accessKey = null, secretKey = null;
		for(int i = 0 ; i < credentials.size(); i++){
			for(String[] pair : credentials.get(i)){
				if(pair[0].equalsIgnoreCase("driver.type")){
					type = pair[1];
				}else if(pair[0].equalsIgnoreCase("driver.id")){
					driverId = pair[1];
				}else if(pair[0].equalsIgnoreCase("accessKey")){
					accessKey = pair[1];
				}else if(pair[0].equalsIgnoreCase("secretKey")){
					secretKey = pair[1];
				}
			}
			drivers[i] = DriversFactory.getDriver(type, driverId, accessKey, secretKey);
		}
	}	
	this.manager = new DepSkySManager(drivers, this, keyLoader);
	this.replies = new HashMap<Integer, CloudRepliesControlSet>();
	this.N = drivers.length;
	this.F = 1;
	this.encoder = new ReedSolEncoder(2, 2, 8);
	this.decoder = new ReedSolDecoder(2, 2, 8);

	if(!startDrivers()){
		System.out.println("Connection Error!");
	}
  }
```

The second step is create too many `src.depskys.core.DepSkySDataUnit` objects as you want. Each object of this type represents our storage model. Concretely, a `src.depskys.core.DepSkySDataUnit` refers to an object that have associated one metadata file and all the versions written to it. The example bellow illustrate it.

```
  exampleFilemetadata
  exampleFilevalue1004
  exampleFilevalue2004
  exampleFilevalue3004
  ...
```

Each `DepSkySDataUnit` object contains information about the protocol used to replicate the data, the metadata information, the written versions, etc. Furthermore, each one of these objects (by that we mean all the files associated with it) can be stored in a different bucket. There are two ways to create a `DepSkySDataUnit` object.
The first example below (1) will write to a container named regId (which will contain regIdmetadata and regIdvalue files) inside a default bucket of DepSky. Using the second example a user is able to specify the bucket where the data will be stored.

```
  (1)
  public DepSkySDataUnit(String regId) {
  ...
  
  (2)
  public DepSkySDataUnit(String regId, String bucketName) {
  ...
```

After creating a `DepSkySDataUnit`object, you need to specify what protocol will be used to replicate the data that will be written in this container. By default, each {{{DepSkySDataUnit}} object will use DepSky-A (data is replicated in clear\_text). To use one of the others three protocols follow the code below.

```
  DepSkySDataUnit dataUnit = new DepSkySDataUnit("container");
  dataUnit.setUsingPVSS(true); //to use DepSky-CA
  dataUnit.setUsingErsCodes(true); //to use only erasure codes
  dataUnit.setUsingSecSharing(true); //to use only secret sharing
```

When you want to perform operations in the `LocalDepSkySClient` object (read, write, etc) you have to use a `DepSkySDataUnit` object.

## **Write** ##

When you want to use the write operation, you have to pass the DepSkySDataUnit object for which you want to write and the data to be written. As we can see below, this operation return a `byte[]`. This `byte[]` is a SHA-1 hash of the written data. This hash must be saved by the client if he want to use the read matching operation (see bellow).

```
  public synchronized byte[] write(DepSkySDataUnit reg, byte[] value) throws Exception {
  ...
```


## **Read** ##

To use this operation, you only have as argument the `DepSkySDataUnit` object. This operation will read the last version written to this `DepSkySDataUnit`.

```
  public synchronized byte[] read(DepSkySDataUnit reg) throws Exception {
  ...
```

## **Read Matching** ##

This operation have the function of read a old version of a given `DepSkySDataUnit`. To do that you have to pass a `byte[]` containing the hash of the version you want to read. This hash is the one returned by the write operation.

```
  public synchronized byte[] readMatching(DepSkySDataUnit reg, byte[] hashMatching) throws Exception{
  ...
```

## **Delete** ##

The delete operation will delete all the files associated with the given `DepSkySDataUnit`, that includes all the versions written and the metadata file.

```
  public synchronized void deleteContainer(DepSkySDataUnit reg) throws Exception{
  ...
```

## **SetAcl** ##

The setacl operation will change the permissions of a specified `DepSkySDataUnit`. Specifically, it will change the permissions of the bucket where the objects are stored, as well as the permissions of the objects within it. For do that we have to share the bucket in the four used clouds (once the data is replicated among them). The protocols to share a bucket in the used clouds can be found [here](http://www.navigators.di.fc.ul.pt/w2/img_auth.php/d/de/Oliveira2014DIHC.pdf).

```
  public synchronized LinkedList<Pair<String, String[]>> setAcl(DepSkySDataUnit reg, String permission,                                      
     LinkedList<Pair<String, String[]>> cannonicalIds) throws Exception {
  ...
```

The operation receives 3 arguments. The first corresponds to the `DepSkySDataUnit` that will be shared. The second specifies the permission that other users will have to access the specified `DepSkySDataUnit`. It can be "r" for read, "w" for write, and "rw" for read and write. The last field has information about the user who will have access to the shared resource. This last field must be constructed following the example below where each line represent an entry in the LinkedList (which is a Pair<String, String[.md](.md)>).

```
 -> <"AMAZON-S3", [canonicalId]>
 -> <"GOOGLE-STORAGE", [email]>
 -> <"RACKSPACE", [name, email]>
 -> <"WINDOWS-AZURE", []>
```

For Amazon S3, the grantee user can find the canonicalId in the same page of the access credential (see the beginning of this page). For the other clouds, the information is quite intuitive. For Google Storage is only need the email of the grantee (must be a gmail account). For RackSpace the name and the grantee. Finally, for Windows Azure nothing is needed (see this [paper](http://www.navigators.di.fc.ul.pt/w2/img_auth.php/d/de/Oliveira2014DIHC.pdf)).

This operation returns a `LinkedList<Pair<String, String[]>>` with the same organization of the one given as argument. This list must be given to the grantee user, as well as the name of the `DepSkySDataUnit` in order he can access the shared resource. But first the user who is sharing must add to it some information. More specifically, he must add to the AMAZON-S3 pair his own cannonicalID, and to the GOOGLE-STORAGE pair his email.

Once the grantee user have this list with he, he can use it in the other operations (read, write, delete) to operate on the shared bucket.