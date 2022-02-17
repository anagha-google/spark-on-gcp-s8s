# About


## 1.0. Variables

PROJECT_KEYWORD="vajra"  

ORG_ID=akhanolkar.altostrat.com                              
ORG_ID_NBR=236589261571
ADMINISTRATOR_UPN_FQN=admin@$ORG_ID 

SVC_PROJECT_NBR=481704770619                           
SVC_PROJECT_ID=dataproc-playground-335723                     
              
LOCATION=us-central1

SVC_PROJECT_UMSA="$PROJECT_KEYWORD-sa"
SVC_PROJECT_UMSA_FQN=$SVC_PROJECT_UMSA@$SVC_PROJECT_ID.iam.gserviceaccount.com

SPARK_GCE_CLUSTER_NM=$PROJECT_KEYWORD-gce
SPARK_GKE_CLUSTER_NM=$PROJECT_KEYWORD-gke
SPARK_SERVERLESS_NM=$PROJECT_KEYWORD-s8s

SPARK_GCE_CLUSTER_BUCKET=gs://$SPARK_GCE_CLUSTER_NM-$SVC_PROJECT_NUMBER
SPARK_GKE_CLUSTER_BUCKET=gs://$SPARK_GKE_CLUSTER_NM-$SVC_PROJECT_NUMBER
SPARK_SERVERLESS_BUCKET=gs://$SPARK_SERVERLESS_NM-$SVC_PROJECT_NUMBER

PERSISTENT_HISTORY_SERVER_NM=$PROJECT_KEYWORD-sphs
PERSISTENT_HISTORY_SERVER_BUCKET=gs://$PROJECT_KEYWORD-sphs-bucket

DATAPROC_METASTORE_SERVICE_NM=$PROJECT_KEYWORD-dpms

VPC_PROJ_ID=$SVC_PROJECT_ID        
VPC_PROJ_ID=$SVC_PROJECT_NBR  

VPC_NM=$PROJECT_KEYWORD-vpc
SPARK_GCE_CLUSTER_SUBNET=$SPARK_GCE_CLUSTER_NM-snet
SPARK_GKE_CLUSTER_SUBNET=$SPARK_GKE_CLUSTER_NM-snet
SPARK_SERVERLESS_SUBNET=$SPARK_SERVERLESS_NM-snet

ADMIN_PUB_IP=98.222.97.10/32




## 2.0. Enable APIs

A tad excesive...
```
gcloud services enable dataproc.googleapis.com
gcloud services enable orgpolicy.googleapis.com
gcloud services enable compute.googleapis.com
gcloud services enable container.googleapis.com
gcloud services enable containerregistry.googleapis.com
gcloud services enable composer.googleapis.com
gcloud services enable monitoring.googleapis.com 
gcloud services enable cloudtrace.googleapis.com 
gcloud services enable clouddebugger.googleapis.com 
gcloud services enable bigquery.googleapis.com 
gcloud services enable storage.googleapis.com
gcloud services enable cloudfunctions.googleapis.com
gcloud services enable pubsub.googleapis.com
gcloud services enable dns.googleapis.com
gcloud services enable sqladmin.googleapis.com
gcloud services enable vpcaccess.googleapis.com

```

## 2.0. Update Organization Policies

### 2.a. Relax require OS Login
```
rm os_login.yaml

cat > os_login.yaml << ENDOFFILE
name: projects/${SVC_PROJECT_ID}/policies/compute.requireOsLogin
spec:
  rules:
  - enforce: false
ENDOFFILE

gcloud org-policies set-policy os_login.yaml 

rm os_login.yaml
```

### 2.b. Disable Serial Port Logging

```
rm disableSerialPortLogging.yaml

cat > disableSerialPortLogging.yaml << ENDOFFILE
name: projects/${SVC_PROJECT_ID}/policies/compute.disableSerialPortLogging
spec:
  rules:
  - enforce: false
ENDOFFILE

gcloud org-policies set-policy disableSerialPortLogging.yaml 

rm disableSerialPortLogging.yaml
```

### 2.c. Disable Shielded VM requirement

```
shieldedVm.yaml 

cat > shieldedVm.yaml << ENDOFFILE
name: projects/$SVC_PROJECT_ID/policies/compute.requireShieldedVm
spec:
  rules:
  - enforce: false
ENDOFFILE

gcloud org-policies set-policy shieldedVm.yaml 

rm shieldedVm.yaml 
```

### 2.d. Disable VM can IP forward requirement

```
rm vmCanIpForward.yaml

cat > vmCanIpForward.yaml << ENDOFFILE
name: projects/$SVC_PROJECT_ID/policies/compute.vmCanIpForward
spec:
  rules:
  - allowAll: true
ENDOFFILE

gcloud org-policies set-policy vmCanIpForward.yaml

rm vmCanIpForward.yaml
```

### 2.e. Enable VM external access

```
rm vmExternalIpAccess.yaml

cat > vmExternalIpAccess.yaml << ENDOFFILE
name: projects/$SVC_PROJECT_ID/policies/compute.vmExternalIpAccess
spec:
  rules:
  - allowAll: true
ENDOFFILE

gcloud org-policies set-policy vmExternalIpAccess.yaml

rm vmExternalIpAccess.yaml
```

### 2.f. Enable restrict VPC peering

```
rm restrictVpcPeering.yaml

cat > restrictVpcPeering.yaml << ENDOFFILE
name: projects/$SVC_PROJECT_ID/policies/compute.restrictVpcPeering
spec:
  rules:
  - allowAll: true
ENDOFFILE

gcloud org-policies set-policy restrictVpcPeering.yaml

rm restrictVpcPeering.yaml
```

## 3.0. Create a User Managed Service Account

gcloud iam service-accounts create ${SVC_PROJECT_UMSA} \
    --description="User Managed Service Account for the $PROJECT_KEYWORD Service Project" \
    --display-name=$SVC_PROJECT_UMSA 

## 4.0. Create VPC, Subnets and Firewall Rules

## 4.a. Create VPC

```
gcloud compute networks create $SHARED_VPC_NETWORK_NM \
--project=$SVC_PROJECT_ID \
--subnet-mode=custom \
--mtu=1460 \
--bgp-routing-mode=regional
```

## 4.b. Create subnet for Dataproc - GCE

```
gcloud compute networks subnets create $SPARK_GCE_CLUSTER_SUBNET \
 --network $VPC_NM \
 --range $10.0.0.0/16 \
 --region $LOCATION \
 --enable-private-ip-google-access \
 --project $SVC_PROJECT_ID 
```

## 4.c. Create subnet for Dataproc - GKE

```
gcloud compute networks subnets create $SPARK_GKE_CLUSTER_SUBNET \
 --network $VPC_NM \
 --range $10.0.3.0/16 \
 --region $LOCATION \
 --enable-private-ip-google-access \
 --project $SVC_PROJECT_ID 
```

## 4.d. Create subnet for Dataproc - S8S


```
gcloud compute networks subnets create $SPARK_SERVERLESS_SUBNET \
 --network $VPC_NM \
 --range $10.0.6.0/16 \
 --region $LOCATION \
 --enable-private-ip-google-access \
 --project $SVC_PROJECT_ID 
```


## 5.0. Create staging buckets for clusters

```
gsutil mb -p $SVC_PROJECT_ID -c STANDARD -l $LOCATION -b on $SPARK_GCE_CLUSTER_BUCKET
gsutil mb -p $SVC_PROJECT_ID -c STANDARD -l $LOCATION -b on $SPARK_GKE_CLUSTER_BUCKET
gsutil mb -p $SVC_PROJECT_ID -c STANDARD -l $LOCATION -b on $SPARK_SERVERLESS_BUCKET

gsutil mb -p $SVC_PROJECT_ID -c STANDARD -l $LOCATION -b on $PERSISTENT_HISTORY_SERVER_BUCKET

```

## 6.0. Create common Persistent Spark History Server

Docs: https://cloud.google.com/dataproc/docs/concepts/jobs/history-server

```
gcloud dataproc clusters create $PERSISTENT_HISTORY_SERVER_NM \
    --region=$LOCATION \
    --image-version=1.4-debian10 \
    --enable-component-gateway \
    --properties='dataproc:job.history.to-gcs.enabled=true,
spark:spark.history.fs.logDirectory=gs://$PERSISTENT_HISTORY_SERVER_BUCKET/fs-logs/spark-job-history,
spark:spark.eventLog.dir=gs://$PERSISTENT_HISTORY_SERVER_BUCKET/event-logs/spark-job-history,
mapred:mapreduce.jobhistory.done-dir=gs://$PERSISTENT_HISTORY_SERVER_BUCKET/event-logs/mapreduce-job-history/done,
mapred:mapreduce.jobhistory.intermediate-done-dir=gs://$PERSISTENT_HISTORY_SERVER_BUCKET/fs-logs/mapreduce-job-history/intermediate-done'
```

## 7.0. Create Dataproc Metastore Service




