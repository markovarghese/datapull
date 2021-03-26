# Test writing dataset with UUID in filesystem to MongoDB

## What is tested?
1. Data can be read from a CSV file in the filesystem
1. Data can be written to an SSL-enabled MongoDB cluster with authentication
   

1. DataPull will create the MongoDB collection if it doesn't exist already
1. DataPull will write UUIDs to MongoDB in MongoDB's UUID datatype , if DataPull's uuidToBinary() Spark SQL function is used

## Pre-requisites

1. Confirm DataPull can run locally, by following the instructions under "Confirm DataPull can run locally" in the [parent folder's README](../README.md)
1. Nothing (usually MongoDB) running on the host machine's port 27017
1. Robo3T installed on the host machine

## Steps

1. In terminal, get to this test's folder
    ```shell
    cd ../manual-tests/filesystem_dataset_with_uuids_to_mongodb/
    ```
1. Start a dockerised mongodb cluster
    ```shell
   cd mongodb_server
   chmod +x install.sh
   ./install.sh
   cd ..
   
1. Build an image for a dockerised spark server, that has the intermediate cert to connect to MongoDB
    ```shell
    docker build -f ./docker_spark_server/Dockerfile -t expedia/spark3.0.2-scala2.12-hadoop3.3.0 ./docker_spark_server
    ```
1. Run DataPull to copy data from a sample dataset in the filesystem, to a collection `testcollection` in the database `testdb` in the dockerised mongodb server
    ```shell
    docker run --network mongo_network -v $(pwd)/../../core/:/core -v $(pwd):/core/manualtestfolder -w /core -it --rm exped
ia/spark3.0.2-scala2.12-hadoop3.3.0 spark-submit  --deploy-mode client --conf "spark.driver.extraJavaOptions=-Djavax.net.ssl.trustStore=/etc/ssl/certs/java/cacerts" --conf "spark.executor.extraJavaOptions=-Djavax.net.ssl.trustStore=
/etc/ssl/certs/java/cacerts" --packages org.apache.spark:spark-sql_2.12:3.0.2,org.apache.spark:spark-avro_2.12:3.0.2,org.apache.spark:spark-sql-kafka-0-10_2.12:3.0.2 --class core.DataPull target/DataMigrationFramework-1.0-SNAPSHOT-j
ar-with-dependencies.jar manualtestfolder/datapull_input.json local
    ```
1. Open Robo3T, and connect to the mongodb server by creating a new connection with
    1. Address: localhost:27017
    1. TLS
       1. Check "Use TLS Protocol"
       1. Authentication Method: Use CA Certificate
       1.  CA Certificate: Location of manual-tests/filesystem_dataset-with_uuids_to_mongodb/mongodb_server/test-ca.pem 
1. Using Robo3T, assert that
    1. There exists a database named `testDb`
    1. There exists a collection named `testcollection` within `testDb`
    1. TODO: fix The collection `testcollection` has documents that include a UUID field `uuidfield` with valid UUID values
1. If the previous step is asserted, then this test is successful; else the test has failed. On failure, please report the failure to the DataPull project team. 

### Cleanup

In terminal, run 
```shell
docker-compose -f ./mongodb_server/docker-compose.yml down -v
```