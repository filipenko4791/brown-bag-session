Testumgebung für die Brown Bag Session am 17.12.2020

tips: https://labs.play-with-docker.com
https://hub.docker.com/

Vorbereitung: 
Docker Desktop for Windows or Mac
Docker Hub

docker login

1. Create a Dockerfile
#Basisimage notwendig abhängig von Software
#im Dockerfile als Instruction enthalten

FROM nginx:alpine
COPY . /usr/share/nginx/html

2. Build Docker Image
# Ergebnis ist ein Docker Image dass gestartet werden kann und die App zum laufen bringt
docker build -t <build-directory> / -t ist für lesbaren Namen
docker build -t brownbag-image:v1

docker images

3. Run
# Jeder Container ist eine Sandbox für die App
# Jeder Container der gestartet  wird benötigt die entsprechenden Freigaben
docker run --name brownbag-web -d -it -p 80:80 brownbag:1.0

4.Push Image to Docker Hub

docker tag bb38976d03cf filipenko23/brownbag-session:tagname
docker push filipenko23/brownbag-session:tagname

Step 1
Aufbauen einer EC2 Instanz
    
AMI	Ubuntu Server 18.04 LTS (HVM), SSD Volume Type
Instance Type	t3.micro (2 vCPU, 1GB Memory)
Storage	8 GB (gp2)
Tags	– Key: Name
– Value: Minikube
Security Group	Name: Minikube Security Group
– SSH, 0.0.0.0/0
Later we will be editing this.
Key Pair	Create your own keypair.
You will need this to SSH to your EC2 Instance

2. SSH into your created EC2 Instance using your keypair.
ssh ubuntu@<ipv4_public_ip> -i <keypair>.pem
Bspw. ssh ubuntu@3.122.252.171  -i /Users/ptiede/Documents/Secrets/BrownBagSession.pem
 Fix for WARNING: UNPROTECTED PRIVATE KEY FILE! 
 sudo chmod 600 /path/to/my/key.pem
 sudo chmod 755 ~/.ssh
 
3. Install kubectl

curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl

chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl

4. Install Docker
sudo apt-get update && \
    sudo apt-get install docker.io -y
    
docker version
    
5. Install Minikube
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/

minikube version

Running Minkube on EC2

sudo -i
sudo apt-get install -y conntrack
minikube start --vm-driver=none
minikube status
minikube dashboard   // Access Remote  minikube dashboard --url && ssh -i <LOCATION TO SSH PRIVATE KEY> -L <LOCAL PORT>:localhost:<REMOTE PORT ON WHICH MINIKUBE DASHBOARD IS RUNNING> user-name@IP
$ sudo ssh -i ~/.ssh/id_rsa -L 8081:localhost:36525 shubham@40.77.75.58


kubectl create deployment brownbag-session --image=filipenko23/brownbag-session:brownsession-webpage
kubectl expose deployment brownbag-session --type=NodePort --port=8080
kubectl get services


3.122.252.171:30268

kubectl create brownbag-session filipenko23/brownbag-session:brownsession-webpage --port=8080
kubectl expose deployment brownbag-session3 --type=NodePort


