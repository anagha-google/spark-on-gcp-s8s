# About

This module covers running **serverless batch jobs** with very basic examples. It covers how to use a persistent Spark History Server and also a common Dataproc Metastore Service.

Lets get started...

## Prerequisites

Completion of the foundational setup module.

## 1. Variables

Modify the varibles below as applicable for your environment and run the same in the cloud shell on the cloud console-

```
PROJECT_KEYWORD="vajra"  

ORG_ID=akhanolkar.altostrat.com                              
ORG_ID_NBR=236589261571
ADMINISTRATOR_UPN_FQN=admin@$ORG_ID 

SVC_PROJECT_NBR=481704770619                           
SVC_PROJECT_ID=dataproc-playground-335723   

#Your public IP address, to add to the firewall
OFFICE_CIDR=98.222.97.10/32
              
LOCATION=us-central1

SVC_PROJECT_UMSA="$PROJECT_KEYWORD-sa"
SVC_PROJECT_UMSA_FQN=$SVC_PROJECT_UMSA@$SVC_PROJECT_ID.iam.gserviceaccount.com


SPARK_SERVERLESS_NM=$PROJECT_KEYWORD-s8s
SPARK_SERVERLESS_BUCKET=gs://$SPARK_SERVERLESS_NM-$SVC_PROJECT_NBR


PERSISTENT_HISTORY_SERVER_NM=$PROJECT_KEYWORD-sphs
PERSISTENT_HISTORY_SERVER_BUCKET=gs://$PERSISTENT_HISTORY_SERVER_NM-$SVC_PROJECT_NBR

DATAPROC_METASTORE_SERVICE_NM=$PROJECT_KEYWORD-dpms

VPC_PROJ_ID=$SVC_PROJECT_ID        
VPC_PROJ_ID=$SVC_PROJECT_NBR  

VPC_NM=$PROJECT_KEYWORD-vpc
SPARK_SERVERLESS_SUBNET=$SPARK_SERVERLESS_NM-snet
SPARK_CATCH_ALL_SUBNET=$PROJECT_KEYWORD-misc-snet
```

<br><br>
<hr>

## 2. Run a SparkPi job

From 

