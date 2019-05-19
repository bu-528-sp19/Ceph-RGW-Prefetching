COSBench - Cloud Object Storage Benchmark
=========================================

COSBench is a benchmarking tool to measure the performance of Cloud Object Storage services. Object storage is an 
emerging technology that is different from traditional file systems (e.g., NFS) or block device systems (e.g., iSCSI).
Amazon S3 and Openstack* swift are well-known object storage solutions.

COSBench now supports OpenStack* Swift and Amplidata v2.3, 2.5 and 3.1, as well as custom adaptors.

Why Modify COSBench? (COSBench + Range Request & Prefetch)
--------------------
As part of our [project](https://github.com/bu-528-sp19/Ceph-RGW-Prefetching) we implemented a simple prefetching system for [Ceph](https://github.com/ceph). We use COSBench to evaluate the performance of Ceph with and without the prefetching system. The prefetching is triggered when there is a prefetch header introduced on GET request for an object on or when there is range request in bytes for an object in S3. COSBench does not support range request or sending customized headers on S3. So, we modified COSBench to meet our needs.

How to Invoke Range Request?
----------------------------
Populate the *config* attribute of the *storage* element with *is_range_request*, *file_length* and *chunk_length*. Here is an example for a workstage:
~~~~
<workstage>
    <work>
        <storage type="s3" config="is_range_request=true;file_length=<file_length>;chunk_length=<chunk_length>;path_style_access=true" />
        <operation type="read" />
    </work>
</workstage>
~~~~
* **is_range_request=true** makes usual READ operation not to read whole file instead it only reads the chunk of the file
* **file_length** is long number which indicated the size of the file in bytes
* **chunk_length** is long number where you indicate how many bytes you want to read from each file in the buckets
> chunk_length < file_length, so what algorithm does is it will randomly request for chunk_length bytes between 0 and file_length - chunk_length

Please refer to [s3-config-range-sample.xml](https://raw.githubusercontent.com/bissenbay/cosbench/release/conf/s3-config-range-sample.xml) for a complete workload configuration.
In this example, we are creating 1 bucket, 10 objects of size 15MB. We are running RANGE workload for 5MB.

How to Invoke Prefetch?
------------------------
Populate the *config* attribute of the *storage* element with *is_range_request*, *file_length* and *chunk_length*. Here is an example for a workstage:
~~~~
<workstage>
    <work>
        <storage type="s3" config="is_prefetch=true;path_style_access=true" />
        <operation type="read" />
    </work>
</workstage>
~~~~
* **is_prefetch=true** makes usual READ operation not to read file at all. *This is only related to our project, so prefetch would send the user 200 OK status and start copying the object to Rados Gateway*

Please refer to [s3-config-prefetch-sample.xml](https://raw.githubusercontent.com/bissenbay/cosbench/release/conf/s3-config-prefetch-sample.xml) for a complete workload configuration.
In this example, we are creating 1 bucket, 10 objects of size 4MB. We are running PREFETCH workload for 4MB.
##### Import Notice
* Make sure to add the flag *-Dcom.amazonaws.services.s3.disableGetObjectMD5Validation=true* to **cosbench-start.sh**. Otherwise, you will face the following error when trying to add an object to the bucket *com.intel.cosbench.api.storage.StorageException: com.amazonaws.services.s3.model.AmazonS3Exception: null (Service: Amazon S3; Status Code: 403; Error Code: SignatureDoesNotMatch;*
* Make sure **path_style_access=true** is introduced in the config attribute. This also might lead to erros when working with S3 PUT operations.


Installation & Usage
--------------------
#### Download
* Obtain installation package for pre-release v0.4.2.c4 from [github (Releases)](https://github.com/intel-cloud/cosbench/releases) and unzip it under the /home directory on the node.
    ```
    wget https://github.com/intel-cloud/cosbench/archive/v0.4.2.c4.zip
    unzip 0.4.2.c4.zip
    cd 0.4.2.c4
    chmod +x *.sh
    ```
    ##### Resons why not to use the latest release
    * The latest release v0.4.2 is broken
    * If v0.4.2 is used, there will be an error in *controller-boot.log*: "Could not find or load main class org.eclipse.equinox.launcher.Main"
    * ./start-all.sh will run forever printing ".Ncat: Connection refused." repeatedly which is open issue [#386](https://github.com/intel-cloud/cosbench/issues/386)
    * The suggested solution at [#240](https://github.com/intel-cloud/cosbench/issues/240) changing TOOL_PARAMS="-i 0" to TOOL_PARAMS="" in *cosbench-start.sh* does not solve the issue
#### Installation
* Install dependencies
    ```
    sudo yum install -y java-1.8.0-openjdk
    sudo yum install nc
    ```
#### Running on a Single Node 
* Start COSBench controller and COSBench driver
    ```
    ./start-all.sh
    ```
* Start COSBench Drivers
    ```
    ./start-driver.sh
    ```
* Start COSBench Controller
    ```
    ./start-controller.sh
    ```

Please refer to [COSBench installation](https://github.com/ekaynar/Benchmarks/tree/master/cosbench) for additional instructions.
Please refer to "COSBenchUserGuide.pdf" for details.

* Submitting a workload
    ```
    ./cli.sh submit <path_to_xml_file>
    ```
* **path_to_xml_file** is the path for the configuration file

Set Up Development Environment
-----------------------------
1. Download [Eclipse Installer](https://www.eclipse.org/downloads/)
2. Install **Eclipse IDE for Enterprise Java Developers**, not Eclipse IDE for Java Developers
3. Download COSBench as mentioned above and import to Eclipse according to the instructions below
    ##### Reason
    * Eclipse IDE for Java does not have Plug-in Development in Preferences
    
    ##### Import Notice
    COSBench is composed of independent projects. So, if you want to make a specific change for some project like in our case (we only modified cosbench-s3 project)
    1. Make sure that your project can be built and there is no errors
    2. Select the project and click "Export -> Plug-in Development -> Deployable plugins and fragments"
    3. Set the "Directory" to "dist\osgi" folder
    4. At "dist\osgi\plugins" folder, there will be generated a .jar file
    5. Stop COSBench *./stop-all.sh*
    6. Copy the generated .jar file to the node where COSBench installed to the directory osgi/plugins and replace it
    7. Run COSBench *./start-all.sh* to test the changes you made

Please refer to [Setting Up Development Environment](https://github.com/ekaynar/Benchmarks/blob/master/cosbench/BUILD.MD) for additional instructions.
If a build from source code is needed, please refer to BUILD.md for details.

Licensing
---------

a) Intel source code is being released under the Apache 2.0 license.

b) Additional libraries used with COSBench have their own licensing; refer to 3rd-party-licenses.pdf for details.


Distribution Packages
---------------------

Please refer to "DISTRIBUTIONS.md" to get the link for distribution packages.

Adaptor Development
-------------------
If needed, adaptors can be developed for new storage services; please refer to "COSBenchAdaptorDevGuide.pdf" for details.

Resources
---------

Wiki: (https://github.com/intel-cloud/cosbench/wiki)

Issue tracking: (https://github.com/intel-cloud/cosbench/issues)

Mailing list: (http://cosbench.1094679.n5.nabble.com/)


Other related projects
----------------------
COSBench-Workload-Generator: (https://github.com/giteshnandre/COSBench-Workload-Generator)

COSBench-Plot: (https://github.com/icclab/cosbench-plot)