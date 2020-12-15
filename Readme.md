# Testumgebung für die Brown Bag Session am 17.12.2020

## Nützliche Links:
* Git repository https://github.com/filipenko4791/brown-bag-session.git
* Docker Registry docker pull filipenko23/brownbag-session
* Getting Started with Docker https://docs.docker.com/get-started/
* Play with Docker https://labs.play-with-docker.com
* Getting Started with Kubernetes https://kubernetes.io/docs/setup/
* Container Transformation Webseite https://blog.direkt-gruppe.de/container-transformation
* Running Minikube in AWS EC2 (Ubuntu) https://www.radishlogic.com/kubernetes/running-minikube-in-aws-ec2-ubuntu/

## Vorbereitung: 
* Docker Desktop for Windows or Mac installieren
* Docker Hub Account anlegen und einlogen
* ggf. GitHub Account

### 1. Login Docker Hub über Terminal
`docker login`
### 2. Docker  Version prüfen
`docker --version`

### 3. Projekt aus GitHub clonen
`git clone https://github.com/filipenko4791/brown-bag-session.git`

## Schritt 1 - Docker Container bauen

### 1. Check Dockerfile
* Basisimage (FROM) ist notwendig und abhängig von verwendeten Software
* im Dockerfile sind Anweisungen enthalten
* TODO: Dockerfile mit folgendem Inhalt erstellen:

```
FROM nginx:alpine
COPY . /usr/share/nginx/html

```

### 2. Build Docker Image
* Ergebnis ist ein Docker Image dass gestartet werden kann und die App zum laufen bringt
* Du musst dich im Directory befinden

`docker build -t brownbag:1.0 . 
docker images`

### 3. Run
* Jeder Container ist eine Sandbox für die App
* Jeder Container der gestartet  wird benötigt die entsprechenden Freigaben

docker run --name brownbag-session -d -it -p 80:80 brownbag-image:1.0

### 4. Erstelltes Image in Docker Hub pushen
* Erstelltes Image in die Registry pushen
* Dabei einen neuen Tag für die Zielregistry vergeben

docker tag bc1c3bf99406 filipenko23/brownbag-session:brownbag-webpage
docker push filipenko23/brownbag-session:brownbag-webpage

## Schritt 2 - Aufbauen einer EC2-Instanz mit Docker und Minikube

### 1. EC2-Instanz erstellen
* AMI: Ubuntu Server 18.04 LTS (HVM), SSD Volume Type
* Instance Type: t3.micro (2 vCPU, 1GB Memory)
* Storage: 8 GB (gp2)
* Tags: – Key: Name, – Value: Minikube
* Security Group: Name: Minikube Security Group, – SSH, 0.0.0.0/0 (wird später noch geändert)
* Key Pair: Eigenes Schlüsselpaar erstellen, wird verwendet um SSH-Verbindung zu EC2-Instanz zu erstellen.

### 2. SSH-Verbindung zur erstellten EC2-Instanz mit Schlüsselpaar herstellen

ssh ubuntu@<ipv4_public_ip> -i <keypair>.pem
 
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

Step 3 
Minkube auf EC2-Instanz ausführen
1. Root User werden
sudo -i

2. Minikube starten
minikube start --vm-driver=none
Fix bei Fehler: sudo apt-get install -y conntrack

3. Status von Minikube checken
minikube status

4. Unseren Container starten
kubectl create deployment brownbag-session --image=filipenko23/brownbag-session:brownbag-webpage

5. Unseren Service starten 
kubectl expose deployment brownbag-session --type=NodePort --port=8080

6. Port des Containers ausfindig machen
kubectl get services

7. Sicherheitsgruppe der EC2-Instanz anpassen

EC2 >> (Network & Security) Security Groups >> Minikube Security Group >> Ingress

Type	Custom TCP Rule
Protocol	TCP
Port Range	30263 (the port given to you by the kubectl get services command)
Source	Custom
0.0.0.0/0 (Accessible via the internet)

8. Den Container via der EC2-Instanz über den Web Browser aufrufen

&lt;ipv4_public_ip&gt;:&lt;ec2_port&gt;.
bspw. 18.157.79.90:32719





minikube dashboard   // Access Remote  minikube dashboard --url && ssh -i <LOCATION TO SSH PRIVATE KEY> -L <LOCAL PORT>:localhost:<REMOTE PORT ON WHICH MINIKUBE DASHBOARD IS RUNNING> user-name@IP
    minikube dashboard --url && ssh -i /Users/ptiede/Documents/Secrets/BrownBagSession.pem -L 8081:localhost:38923 ubuntu@18.157.79.90
$ sudo ssh -i ~/.ssh/id_rsa -L 8081:localhost:36525 shubham@40.77.75.58



