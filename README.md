#### Deploy kubernetes cluster with jenkins pipeline

First create a cluster using eksctl
##### EKSCTL create cluster command

```
eksctl create cluster \
--name demo-cluster \
--version 1.27 \
--region ca-central-1 \
--nodegroup-name demo-nodes \
--node-type t2.micro \
--nodes 2 \
--nodes-min 1 \
--nodes-max 3
```

Steps we need to configure
#### -- Install kubectl command line tool inside jenkins container, we need to execute kubectl command inside jenkins pipeline

#### -- Install aws-iam-authenticator tool inside jenkins container, cuz we need to authenticate with both Kubernetes cluster and aws account seperately...both of these command line tools needs to be available for jenkins so that they get executed in background when we run our pipeline

#### -- create kubeconfig file to connect to EKS cluster (this is alternative to creating credentials inside jenkins which we use to do using jenkins UI ) 

##### We gonna go inside jenkins container directly and we gonna create kubeconfig file that contains all the information we need inorder to authenticate with aws account using aws-iam-authenticator command line tools and also with eks cluster that we have created

##### On our local machine when we created eks cluster, we saw .kube/config folder got generated so, the config file got automatically installed there, this configures kubectl on our local machine to talk to the eks cluster.. this config file contains all the necessary info for the authentication

##### When we create eks cluster using eksctl tool, it automatically installs aws-iam-authenticator for us.
So we need to do same inside jenkins container so that jenkins can do the same

#### -- Add AWS credentials on jenkins for aws account authentication
so we need to have aws user that we used to create the cluster, each user on aws has accesskey and secretAccessKey, we need this credentials to connect to our cluster

So we need bunch of credentials to connect to aws and cluster running inside there.

#### adjust jenkinsfile to configure eks cluster deployment

Go into Jenkins Container(i installed a jenkins as a container inside the digital ocean droplet)
ssh into the server and 
#### -- docker exec -u 0 -it container_ID bash

### These commands are used in the Deploy to EKS from Jenkins lecture

##### Install kubectl on Jenkins server
 
```sh
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl; chmod +x ./kubectl; mv ./kubectl /usr/local/bin/kubectl
```

##### Install aws-iam-authenticator on Jenkins server

 ```sh
 curl -Lo aws-iam-authenticator https://github.com/kubernetes-sigs/aws-iam-authenticator/releases/download/v0.6.11/aws-iam-authenticator_0.6.11_linux_amd64
 chmod +x ./aws-iam-authenticator
 mv ./aws-iam-authenticator /usr/local/bin
 
```
So we gonna create kubeconfig file manually outside the container on the host on digital ocean droplet and we gonna copy that inside jenkins container


##### Copy config file to Jenkins server

 ```sh
 docker cp config "JENKINS_CONTAINER ID":/var/jenkins_home/.kube/

 ```
 
 #### credentials for aws user (we use admin user)
 kind -- secret text (on jenkins UI)
 
 access_key
 secret_access_key both are available for multi branch pipeline
 
enter into jenkins container as a root user (docker exec -u 0 -it container_ID bash)

install envsubst inside
-- apt-get install gettext-base 

sh 'envsubst < kubernetes/deployment.yaml | kubectl apply -f -' 
this command will take the file, look for syntax of dollar sign and name of variable like $IMAGE_NAME and try to match the name of variable to any env variable defined in the context of jenkinsfile, it will create temporary file with value set and we gonna pipe that temporary file and pass it as a parameter to the next command kubectl apply -f -

#### Create Secret for DockerHub Credentials
when we do kubectl apply -f - it basically fetches private images from docker hub registry so that we need authentication with the registry from inside of kubernetes cluster

###secrets can be created once in namespace or in pipeline or in commandline using kubectl command

--------------------------- using docker hub as a private repository ----------------------------

``` kubectl create secret docker-registry my-registry-key \
    --docker-server=docker.io \
    --docker-username=prashzan \
    --docker-password=docker_password
```

#### -- kubectl get secret

----------------------------- Using aws ECR registry instead of dockerhub -------------------------------

However, we are also pulling this image into the kubernetes cluster, we need a secret that holds this credentials..
    
``` kubectl create secret docker-registry aws-registry-key \
    --docker-server=123456789.dkr.ecr.eu-central-1.amazonaws.com \
    --docker-username=AWS \
    --docker-password=password from aws ecr (aws ecr get-login-password)
```





