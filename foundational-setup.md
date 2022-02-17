# About


## 1.0. Variables

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

SPARK_GCE_CLUSTER_NM=$PROJECT_KEYWORD-gce
SPARK_GKE_CLUSTER_NM=$PROJECT_KEYWORD-gke
SPARK_SERVERLESS_NM=$PROJECT_KEYWORD-s8s

SPARK_GCE_CLUSTER_BUCKET=gs://$SPARK_GCE_CLUSTER_NM-$SVC_PROJECT_NBR
SPARK_GKE_CLUSTER_BUCKET=gs://$SPARK_GKE_CLUSTER_NM-$SVC_PROJECT_NBR
SPARK_SERVERLESS_BUCKET=gs://$SPARK_SERVERLESS_NM-$SVC_PROJECT_NBR


PERSISTENT_HISTORY_SERVER_NM=$PROJECT_KEYWORD-sphs
PERSISTENT_HISTORY_SERVER_BUCKET=gs://$PERSISTENT_HISTORY_SERVER_NM-$SVC_PROJECT_NBR

DATAPROC_METASTORE_SERVICE_NM=$PROJECT_KEYWORD-dpms

VPC_PROJ_ID=$SVC_PROJECT_ID        
VPC_PROJ_ID=$SVC_PROJECT_NBR  

VPC_NM=$PROJECT_KEYWORD-vpc
SPARK_GCE_CLUSTER_SUBNET=$SPARK_GCE_CLUSTER_NM-snet
SPARK_GKE_CLUSTER_SUBNET=$SPARK_GKE_CLUSTER_NM-snet
SPARK_SERVERLESS_SUBNET=$SPARK_SERVERLESS_NM-snet
SPARK_CATCH_ALL_SUBNET=$PROJECT_KEYWORD-misc-snet

OFFICE_CIDR=98.222.97.10/32
```



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
gcloud services enable metastore.googleapis.com

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

## 3.0. Create a User Managed Service Account (UMSA) & grant it requisite permissions

### 3.a. Create UMSA
```
gcloud iam service-accounts create ${SVC_PROJECT_UMSA} \
    --description="User Managed Service Account for the $PROJECT_KEYWORD Service Project" \
    --display-name=$SVC_PROJECT_UMSA 
```
### 3.b. Grant IAM permissions for UMSA

```
gcloud projects add-iam-policy-binding ${SVC_PROJECT_ID} \
    --member=serviceAccount:${SVC_PROJECT_UMSA_FQN} \
    --role=roles/iam.serviceAccountUser
    
gcloud projects add-iam-policy-binding ${SVC_PROJECT_ID} \
    --member=serviceAccount:${SVC_PROJECT_UMSA_FQN} \
    --role=roles/iam.serviceAccountTokenCreator 
    
gcloud projects add-iam-policy-binding $SVC_PROJECT_ID --member=serviceAccount:$SVC_PROJECT_UMSA_FQN \
--role="roles/bigquery.dataEditor"


gcloud projects add-iam-policy-binding $SVC_PROJECT_ID --member=serviceAccount:$SVC_PROJECT_UMSA_FQN \
--role="roles/bigquery.admin"

gcloud projects add-iam-policy-binding $SVC_PROJECT_ID --member=serviceAccount:$SVC_PROJECT_UMSA_FQN \
--role="roles/dataproc.worker"

gcloud projects add-iam-policy-binding $SVC_PROJECT_ID --member=serviceAccount:$SVC_PROJECT_UMSA_FQN \
--role="roles/metastore.admin"

gcloud projects add-iam-policy-binding $SVC_PROJECT_ID --member=serviceAccount:$SVC_PROJECT_UMSA_FQN \
--role="roles/metastore.editor"

```

### 3.c. Grant permissions to the Compute Engine Default Google Managed Service Account

```
COMPUTE_ENGINE_DEFAULT_GMSA=$SVC_PROJECT_NBR-compute@developer.gserviceaccount.com

gcloud projects add-iam-policy-binding $SVC_PROJECT_ID --member=serviceAccount:$COMPUTE_ENGINE_DEFAULT_GMSA \
--role="roles/bigquery.dataEditor"

gcloud projects add-iam-policy-binding $SVC_PROJECT_ID --member=serviceAccount:$COMPUTE_ENGINE_DEFAULT_GMSA \
--role="roles/bigquery.admin"

gcloud projects add-iam-policy-binding $SVC_PROJECT_ID --member=serviceAccount:$COMPUTE_ENGINE_DEFAULT_GMSA \
--role="roles/dataproc.worker"
```


### 3.d. Grant permissions for the lab attendee

```
gcloud iam service-accounts add-iam-policy-binding \
    ${SVC_PROJECT_UMSA_FQN} \
    --member="user:${ADMINISTRATOR_UPN_FQN}" \
    --role="roles/iam.serviceAccountUser"
    
gcloud iam service-accounts add-iam-policy-binding \
    ${SVC_PROJECT_UMSA_FQN} \
    --member="user:${ADMINISTRATOR_UPN_FQN}" \
    --role="roles/iam.serviceAccountTokenCreator"
    

gcloud projects add-iam-policy-binding $SVC_PROJECT_ID --member=user:$ADMINISTRATOR_UPN_FQN \
--role="roles/bigquery.user"

gcloud projects add-iam-policy-binding $SVC_PROJECT_ID --member=user:$ADMINISTRATOR_UPN_FQN \
--role="roles/bigquery.dataEditor"

gcloud projects add-iam-policy-binding $SVC_PROJECT_ID --member=user:$ADMINISTRATOR_UPN_FQN \
--role="roles/bigquery.jobUser"


gcloud projects add-iam-policy-binding $SVC_PROJECT_ID --member=user:$ADMINISTRATOR_UPN_FQN \
--role="roles/bigquery.admin"
```




## 4.0. Create VPC, Subnets and Firewall Rules

## 4.a. Create VPC

```
gcloud compute networks create $VPC_NM \
--project=$SVC_PROJECT_ID \
--subnet-mode=custom \
--mtu=1460 \
--bgp-routing-mode=regional
```

## 4.b. Create subnet & firewall rules for Dataproc - GCE

```
SPARK_GCE_CLUSTER_SUBNET_CIDR=10.0.0.0/16

gcloud compute networks subnets create $SPARK_GCE_CLUSTER_SUBNET \
 --network $VPC_NM \
 --range 10.0.0.0/16 \
 --region $LOCATION \
 --enable-private-ip-google-access \
 --project $SVC_PROJECT_ID 
 
gcloud compute --project=$SVC_PROJECT_ID firewall-rules create allow-intra-$SPARK_GCE_CLUSTER_SUBNET \
--direction=INGRESS \
--priority=1000 \
--network=$VPC_NM \
--action=ALLOW \
--rules=all \
--source-ranges=$SPARK_GKE_CLUSTER_SUBNET_CIDR
 

```

## 4.c. Create subnet & firewall rules for Dataproc - GKE

```
SPARK_GKE_CLUSTER_SUBNET_CIDR=10.2.0.0/16

gcloud compute networks subnets create $SPARK_GKE_CLUSTER_SUBNET \
 --network $VPC_NM \
 --range $SPARK_GKE_CLUSTER_SUBNET_CIDR \
 --region $LOCATION \
 --enable-private-ip-google-access \
 --project $SVC_PROJECT_ID
 
gcloud compute --project=$SVC_PROJECT_ID firewall-rules create allow-intra-$SPARK_GKE_CLUSTER_SUBNET \
--direction=INGRESS \
--priority=1000 \
--network=$VPC_NM \
--action=ALLOW \
--rules=all \
--source-ranges=$SPARK_GKE_CLUSTER_SUBNET_CIDR
```

## 4.d. Create subnet & firewall rules for Dataproc - S8S


```
SPARK_SERVERLESS_SUBNET_CIDR=10.4.0.0/16

gcloud compute networks subnets create $SPARK_SERVERLESS_SUBNET \
 --network $VPC_NM \
 --range $SPARK_SERVERLESS_SUBNET_CIDR  \
 --region $LOCATION \
 --enable-private-ip-google-access \
 --project $SVC_PROJECT_ID 
 
gcloud compute --project=$SVC_PROJECT_ID firewall-rules create allow-intra-$SPARK_SERVERLESS_SUBNET \
--direction=INGRESS \
--priority=1000 \
--network=$VPC_NM \
--action=ALLOW \
--rules=all \
--source-ranges=$SPARK_SERVERLESS_SUBNET_CIDR
```

## 4.e. Create subnet & firewall rules for Dataproc - PSHS & DPMS

```
SPARK_CATCH_ALL_SUBNET_CIDR=10.6.0.0/24

gcloud compute networks subnets create $SPARK_CATCH_ALL_SUBNET \
 --network $VPC_NM \
 --range $SPARK_CATCH_ALL_SUBNET_CIDR \
 --region $LOCATION \
 --enable-private-ip-google-access \
 --project $SVC_PROJECT_ID 
 
gcloud compute --project=$SVC_PROJECT_ID firewall-rules create allow-intra-$SPARK_CATCH_ALL_SUBNET \
--direction=INGRESS \
--priority=1000 \
--network=$VPC_NM \
--action=ALLOW \
--rules=all \
--source-ranges=$SPARK_CATCH_ALL_SUBNET_CIDR
 
```

### 4.f. Grant the office CIDR access

```
gcloud compute firewall-rules create allow-ingress-from-office \
--direction=INGRESS \
--priority=1000 \
--network=$VPC_NM \
--action=ALLOW \
--rules=all \
--source-ranges=$OFFICE_CIDR
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
    --single-node \
    --region=$LOCATION \
    --image-version=1.4-debian10 \
    --enable-component-gateway \
    --properties="dataproc:job.history.to-gcs.enabled=true,spark:spark.history.fs.logDirectory=$PERSISTENT_HISTORY_SERVER_BUCKET/*/spark-job-history,mapred:mapreduce.jobhistory.read-only.dir-pattern=$PERSISTENT_HISTORY_SERVER_BUCKET/*/mapreduce-job-history/done" \
    --service-account=$SVC_PROJECT_UMSA_FQN \
--single-node \
--subnet=projects/$SVC_PROJECT_ID/regions/$LOCATION/subnetworks/$SPARK_CATCH_ALL_SUBNET
```



## 7.0. Create Dataproc Metastore Service

Does not support BYO subnet-
```
gcloud metastore services create $DATAPROC_METASTORE_SERVICE_NM \
    --location=$LOCATION \
    --labels=used-by=all-vajra-clusters \
    --network=$VPC_NM \
    --port=9083 \
    --tier=Developer \
    --hive-metastore-version=3.1.2 \
    --impersonate-service-account=$SVC_PROJECT_UMSA_FQN 
```
