# About

This module covers basics of running Serverless Spark on GCP.

## 1. Variables

```
PROJECT_KEYWORD="vajra"  

ORG_ID=akhanolkar.altostrat.com                              
ORG_ID_NBR=236589261571
ADMINISTRATOR_UPN_FQN=admin@$ORG_ID 

SVC_PROJECT_NBR=481704770619                           
SVC_PROJECT_ID=dataproc-playground-335723                     
              
LOCATION=us-central1

SVC_PROJECT_UMSA="$PROJECT_KEYWORD-sa"
SVC_PROJECT_UMSA_FQN=$SVC_PROJECT_UMSA@$SVC_PROJECT_ID.iam.gserviceaccount.com


SPARK_SERVERLESS_NM=$PROJECT_KEYWORD-s8s
SPARK_SERVERLESS_BUCKET=gs://$SPARK_SERVERLESS_NM-$SVC_PROJECT_NBR


PERSISTENT_HISTORY_SERVER_NM=$PROJECT_KEYWORD-sphs
PERSISTENT_HISTORY_SERVER_BUCKET=gs://$PERSISTENT_HISTORY_SERVER_NM-$SVC_PROJECT_NBR

DATAPROC_METASTORE_SERVICE_NM=$PROJECT_KEYWORD-dpms

VPC_NM=$PROJECT_KEYWORD-vpc
SPARK_SERVERLESS_SUBNET=$SPARK_SERVERLESS_NM-snet

OFFICE_CIDR=98.222.97.10/32
```

```
gcloud dataproc batches submit spark
--project=dataproc-playground-335723
--region=us-central1
--subnet projects/dataproc-playground-335723/regions/us-central1/subnetworks/serverless-snet
--jars=file:///usr/lib/spark/examples/jars/spark-examples.jar
--class org.apache.spark.examples.SparkPi -- 10000
```
