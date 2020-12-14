Testumgebung für die Brown Bag Session am 17.12.2020

tips: https://labs.play-with-docker.com
https://hub.docker.com/

Vorbereitung: 
Docker Desktop for Windows or Mac
Docker Hub


1. Create a Dockerfile
Basisimage notwendig abhängig von Software
im Dockerfile als Instruction enthalten

FROM nginx:alpine
COPY . /usr/share/nginx/html
CMD html /webpage.html

2. Build Docker Image
build  command
Ergebnis ist ein Docker Image dass gestartet werden kann und die App zum laufen bringt
docker build -t <build-directory> / -t ist für lesbaren Namen
docker build -t brownbag-image:v1
docker images

3. Run
Jeder Container ist eine Sandbox für die App
Jeder Container der gestartet  wird benötigt die entsprechenden Freigaben
docker run -d -p 80:80 brownbag:v1
curl docker


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
    
5. Install Minikube
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/

