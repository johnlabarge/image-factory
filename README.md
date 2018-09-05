
To add to an existing project follow the instructions below: 
## 0. Create a project with a billing account attached (OPTIONAL you can also use an existing project). (see also ) 
```sh 
PROJECT=[YOUR PROJECT NAME]
ORG=[YOUR_ORG]
BILLING_ACCOUNT=[YOUR_BILLING_ACCOUNT_NAME]
ZONE=[COMPUTE ZONE YOU WANT TO USE]
gcloud projects create $PROJECT" --organization=$ORG
gcloud beta billing projects link $PROJECT --billing-account=$(gcloud beta billing accounts list | grep $BILLING_ACCOUNT | awk '{print $1}')
gcloud config configurations create -- activate $PROJECT
gcloud config set compute/zone $ZONE
```

# 1. Set the project variable 
PROJECT=$(gcloud config get-value project)

# 2. Enable the required services
gcloud services enable sourcerepo.googleapis.com
gcloud services enable cloudapis.googleapis.com 
gcloud services enable compute.googleapis.com
gcloud services enable servicemanagement.googleapis.com 
gcloud services enable storage-api.googleapis.com  
gcloud services enable cloudbuild.googleapis.com

# Give the Cloud Build user Project Editor permissions 
```sh
CLOUD_BUILD_ACCOUNT=$(gcloud projects get-iam-policy $project --format=flattened | grep @cloudbuild | awk '{print $2}')

gcloud projects add-iam-policy-binding $PROJECT \
  --member $CLOUD_BUILD_ACCOUNT \
  --role roles/editor   
```

# Create the source repository for your image creator 
gcloud source repos create helloworld-image-factory


# Create the build triggers for the image creator source repository 
Go to https://console.cloud.google.com/cloud-build/triggers and create a trigger.  

1. Click "Create Trigger" 
1. Select "Cloud Source Repository" Click "Continue".
1. Select "helloworld-image-factory" anc click "Continue" 
1. Enter "Hello world image factory" for name." 
1. Set the trigger for tag. 
1. Set the build type to cloudbuild.yaml
1. Set the substitution, _IMAGE_FAMILY to centos-7
1. Set the substitution, _IMAGE_ZONE to the zone you want to use $ZONE. 
**To see a list of image families: 
```sh
gcloud compute images list | awk '{print $3}'  | awk '!a[$0]++'
```

# Add the packer cloudbuild image to your project

```sh
project_dir=$(pwd)
cd /tmp
git clone https://github.com/GoogleCloudPlatform/cloud-builders-community.git
cd cloud-builders-community/packer
gcloud builds submit --config cloudbuild.yaml
rm -f /tmp/cloud-builders-community
cd $project_dir
```

# Add your helloworld-image-factory google repository as a remote repository with the name 'google'
```sh 
gcloud init && git config --global credential.https://source.developers.google.com.helper gcloud.sh
git remote add google \
  https://source.developers.google.com/p/$PROJECT/r/helloworld-image-factory
```

# Tag this commit 
```sh 
git tag v0.1
```

# Push the repository and tags to google. 
```sh
git push google master --tags
```

# Create a compute instance for the image in your gcp project 
gcloud compute instances create --image



