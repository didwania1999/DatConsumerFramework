## Data Consumer Framework Prototype
# This is a prototype of Data Consumer Framework!

# Features Added:
Connectivity to Snowflake using JDBC and SCD Type 1 & 2 functionality

# Steps :
- [x] Clone the repository  in IntelliJ Idea
- [x] Add the following pom.xml file in the root project directory
- [x] Use Maven as the build tool (SBT or Gradle not compatible with the drivers used)
- [x] Use Scala 2.11.12 SDK for Global Libraries
- [x] Build the project
- [x] Invalidate caches in case some libraries are not recognised.
# Versions Used:
- [x] Scala 2.11.6
- [x] Spark 2.4
- [x] Hadoop 3.2.1
- [x] (some versions of scala were not compatible with Snowflake drivers)

# Creating a JAR File
Go to the project directory in Powershell run  [mvn package]

# Running Locally:

spark-submit --packages net.snowflake:spark-snowflake_2.11:2.8.6-spark_2.4,net.snowflake:snowflake-jdbc:3.13.2,com.typesafe:config:1.4.1  --master local --class ETLPROCESS.Driver C:\Users\ASUS\IdeaProjects\test\target\DCF.jar  local  application.conf  input.json

# HDInsight

- [x] 1.Create a Spark cluster on Microsoft Azure HDInsight.[It will take 30 mins for the deployment to succeed]
- [x] 2.Upload the jar file in Azure Blob container.
- [x] 3.Connect to the cluster using PuTTY with ssh available in access keys on the Cluster Portal.
- [x] 4.Run the Spark Submit to trigger the job.
- [x] Format for Spark Submit command
spark-submit --packages net.snowflake:spark-snowflake_2.11:2.8.6-spark_2.4,net.snowflake:snowflake-jdbc:3.13.2,com.typesafe:config:1.4.1  --master yarn --deploy-mode client --class ETLPROCESS.Driver  wasbs://intern2021-2021-08-10t09-15-33-363z@intern2021hdistorage.blob.core.windows.net/DCF.jar yarn application.conf input.json 

# Format to access files on Azure Blob Storage:
<code>
  wasbs://<storage container name>@<storage account name>hdistorage.blob.core.windows.net/<filepath>
</code>



## Docker Image Creation
(Powershell in Administrator Mode)

# Powershell 1:
```
docker pull gradiant/spark
docker images 
docker run -it <container id for the spark image>/bin/sh
```


# Powershell 2:
  ```
cd <path to jar file>
docker cp <jar file name> <containerid>:/opt/spark/jars
  ```

# Powershell 3:
  ```
docker commit <containerid> <imagename>
//push image to docker hub
docker push <imagename> <dockerhub-username>/<your_imagename>
 

//push image to Azure Container Registry
docker tag <your imagename>:<versiontag> <container registry url>/<your imagename>:<tag>

docker push <container registry url>/<imagename>:<tag>

```
# Steps to log in to azure in powershell:
  ```
az login
az acr --name <container registry name>
  ```

## Azure Kubernetes Service:

1.Create a Kubernetes cluster on AKS
2.When clicking on Connect icon on the dashboard run the two commands on Powershell
3.Create Service Account
kubectl create serviceaccount spark
kubectl create clusterrolebinding spark-role --clusterrole=edit --serviceaccount=default:spark --namespace=default
4.Run Spark Submit
Token : On the Cluster Portal ->Configurations->Secrets->Default namespace->file with service account name->token
```
spark-submit --master  k8s://https://dcf-dns-8a968e7d.hcp.eastus.azmk8s.io:443 --deploy-mode cluster --jars local:///opt/spark/jars/spark-snowflake_2.11-2.8.6-spark_2.4.jar,local:///opt/spark/jars/snowflake-jdbc-3.13.2.jar,local:///opt/spark/jars/config-1.4.1.jar  --class ETLPROCESS.Driver --conf spark.kubernetes.container.image=intern2021.azurecr.io/scd1:v1 --conf spark.kubernetes.container.runAsUser=1001 --conf spark.kubernetes.namespace=default --conf spark.kubernetes.authenticate.driver.serviceAccountName=spark --conf spark.kubernetes.authenticate.submission.oauthToken=<token>   --conf spark.kubernetes.driver.limit.cores=4 --conf spark.kubernetes.executor.limit.cores=4 --conf spark.executor.instances=4   local:///opt/spark/jars/TESTING-0.1.jar  k8s://https://dcf-dns-8a968e7d.hcp.eastus.azmk8s.io:443  application.conf input.json
  ```

# To check Job Status
  ```
kubectl get pods
kubectl logs <driver pod>
  ```

# Current Challenges:
1.The ‘input.json’ and ‘application.conf’ files are packages inside our jar file.When the same file is given externally it shows error “no configuration setting found for key extract_load”
Even in local when jar file is moved out of project folder same error.


2.Reading files from azure blob storage using the file path
```
 spark.conf.set( "fs.azure.sas.intern2021-2021-08-10t09-15-33-363z.intern2021hdistorage.blob.core.windows.net",        "sp=r&st=2021-08-10T17:46:23Z&se=2021-08-11T01:46:23Z&spr=https&sv=2020-08-04&sr=b&sig=wkGJgLsBkkl5FivppuA3VcHk%2BCnG25bUxAiMqqFWvfs%3D")
df=spark.read.format("org.apache.spark.sql.execution.datasources.csv.CSVFileFormat").option("header", "true").option("inferSchema","true").load(file_path)
```
Maven packages added
https://mvnrepository.com/artifact/org.apache.hadoop/hadoop-azure
https://mvnrepository.com/artifact/com.microsoft.azure/azure-storage

With the above configuration locally and on HDInsight file can be read but the same jar file gives the following error while running on docker container or Kubernetes:-
“No file system found for scheme wasbs”



Thank You !
