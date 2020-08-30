# helm-chart-metricserver-HPA

# Pre-Requisites
    - Install Git
    - Install Maven
    - Install Docker
    - Install EKS-Cluster
    - Install Helm
# Install Java:
    yum install java-1.8.0-openjdk-devel -y
# Install Git:
    yum install git -y
# Install Apache-Maven:
    wget https://mirrors.estointernet.in/apache/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz
    tar xvzf apache-maven-3.6.3-bin.tar.gz

    vi /etc/profile.d/maven.sh
    --------------------------------------------
    export MAVEN_HOME=/opt/apache-maven-3.6.3
    export PATH=$PATH:$MAVEN_HOME/bin
    --------------------------------------------

    source /etc/profile.d/maven.sh
    mvn -version
# Install Docker:
    yum install docker -y
    service docker start
# Clone code from github:
    git clone https://github.com/cloudtechmasters/helm-chart-metricserver-HPA.git
    cd helm-chart-metricserver-HPA/hellospringbooteks
# Build Maven Artifact:
    mvn clean install
# Build Docker image for Springboot Application
    docker build -t cloudtechmasters/hellospringbooteks .
# Docker login
    docker login
# Push docker image to dockerhub
    docker push cloudtechmasters/hellospringbooteks
# Deploy metric-server helm chart
    helm install metrics-server metrics-server
# Note
  We need to do Two changes inside metrics-server
  1. Open values.yaml and "enable hostNetwork as true"
    
    hostNetwork:
      enabled: true
  2. Also add below command args in side templates/metrics-server-deployment.yaml

    - name: metrics-server
      command:
        - /metrics-server
        - --cert-dir=/tmp
        - --logtostderr
        - --secure-port=8443
        - --kubelet-insecure-tls=true
        - --v=2
        - --kubelet-preferred-address-types=InternalIP
# Deploy spring boot application with helm charts
    cd hellospringbooteks
    helm install hellospringbooteks hellospringbooteks
  Note:
    1. Before deploy we need to edit values.yaml file for HPA as show below
  ![image](https://user-images.githubusercontent.com/68885738/91662933-3a136400-eb03-11ea-8de3-7a1c2370e819.png)
    2. Edit container port number, livenessProbe and readinessProbe as shown below
  ![image](https://user-images.githubusercontent.com/68885738/91662981-89f22b00-eb03-11ea-8d43-f2a6f83e702b.png)
# Check 
    kubectl get all
 ![image](https://user-images.githubusercontent.com/68885738/91662392-c6bc2300-eaff-11ea-9b70-684f40bd887e.png)

Here we can check howmuch % of CPU utilizating, and now we need to increase CPU utilization by using following steps
  1. Connect to POD:
    
    kubectl exec -it hellospringbooteks-5794fccb68-k5kw4 -- /bin/bash
  2. To increase CPU utilization give below command:
	  
    for i in 1 2 3 4; do while : ; do : ; done & done
  Now check with below command whether the pods are decreased or not: (Wait 2min to 3min)
    
    kubectl get all
 Now check below image (Replicas increases to 3 from 2)
  ![image](https://user-images.githubusercontent.com/68885738/91662525-b6587800-eb00-11ea-8746-ab84df376531.png)
 Because of HPA pods are created when CPU utilization increased more than 40%
 Now going to Reduce CPU utilization to check whether the pods are reducing accordingly or not:
	  
    for i in 1 2 3 4; do kill %$i; done
 Now check with below command whether the pods are decreased or not: (Wait 2min to 3min)
	
    kubectl get all
  ![image](https://user-images.githubusercontent.com/68885738/91662801-595dc180-eb02-11ea-9592-aac5641658f9.png)

# Check all events
    kubectl get events --all-namespaces
  ![image](https://user-images.githubusercontent.com/68885738/91662832-85794280-eb02-11ea-8765-51d0c464db22.png)
