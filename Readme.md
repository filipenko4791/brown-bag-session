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
```
docker login
```
### 2. Docker  Version prüfen
```
docker --version
```

### 3. Projekt aus GitHub clonen
```
git clone https://github.com/filipenko4791/brown-bag-session.git
```

## Schritt 1 - Docker Container bauen
Die Tags des Images und des Services kannst du frei wählen (brownbag-image, brownbag-session, brownbag-webpage, brownbag-session). Du kannst aber auch die Namen so lassen.
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

```
docker build -t brownbag-image:1.0 . 
docker images
```

### 3. Run
* Jeder Container ist eine Sandbox für die App
* Jeder Container der gestartet  wird benötigt die entsprechenden Freigaben

```
docker run --name brownbag-session -d -it -p 80:80 brownbag-image:1.0
```

### 4. Erstelltes Image in Docker Hub pushen
* Das erstellte Image benötigt einen neuen Tag mit der Zielregistry
* Erst dann kann das erstellte Image in die Registry geschoben werden
* Den Pfad der Registry kannst du im Docker Hub nachschlagen

```
docker tag image-id registrypfad:brownbag-webpage
docker push registrypfad:brownbag-webpage
```

## Schritt 2 - Aufbauen einer EC2-Instanz mit Docker und Minikube

### 1. EC2-Instanz erstellen
* AMI: Ubuntu Server 18.04 LTS (HVM), SSD Volume Type
* Instance Type: t3.micro (2 vCPU, 1GB Memory)
* Storage: 8 GB (gp2)
* Tags: – Key: Name, – Value: Minikube
* Security Group: Name: Minikube Security Group, – SSH, 0.0.0.0/0 (wird später noch geändert)
* Key Pair: Eigenes Schlüsselpaar erstellen, wird verwendet um SSH-Verbindung zu EC2-Instanz zu erstellen.

### 2. SSH-Verbindung zur erstellten EC2-Instanz mit Schlüsselpaar herstellen

```
ssh ubuntu@<ipv4_public_ip> -i <keypair>.pem
```

**Fix für WARNING: UNPROTECTED PRIVATE KEY FILE!**
```
sudo chmod 600 /path/to/my/key.pem
sudo chmod 755 ~/.ssh
```
 
### 3. kubectl installieren

```
curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
```
```
chmod +x ./kubectl
```
```
sudo mv ./kubectl /usr/local/bin/kubectl
```

### 4. Docker installieren

Minikube benötigt Docker. 
```
sudo apt-get update && \
    sudo apt-get install docker.io -y
````
Docker Version prüfen.    
```
docker version
```    

### 5. Install Minikube
```
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
````
### 6. Minikube Version prüfen
```
minikube version
````

## Schritt 3 - Minkube auf EC2-Instanz ausführen

Root User werden.
```
sudo -i
````

### 1. Minikube starten
```
minikube start --vm-driver=none
```

**Fix bei Conntrack-Fehler**
```
sudo apt-get install -y conntrack
```

### 2. Status von Minikube checken
```
minikube status
````

### 3. Unseren Container starten
```
kubectl create deployment brownbag-session --image=registrypfad:brownbag-webpage
```
### 4. Ersten Service starten 
```
kubectl expose deployment brownbag-session --type=NodePort --port=80
```

### 5. Port des Containers ausfindig machen
```
kubectl get services
````

### 6. Sicherheitsgruppe der EC2-Instanz anpassen

* EC2 >> Erstelle EC2-Instanz anwählen >> Sicherheit >> Minikube Security Group >> Regeln für eingehenden Datenverkehr*

* Type: Custom TCP Rule
* Protocol: TCP
* Port Range: 30263 (der Port aus dem Befehl kubectl get services command)
* Source: Custom, 0.0.0.0/0 (Zugriff aus dem Internet)

### 7. Den Container via der EC2-Instanz über den Web Browser aufrufen
```
&lt;ipv4_public_ip&gt;:&lt;ec2_port&gt;.
````

Bspw. 18.157.79.90:31559



#### Minikube Dashboard aufrufen
```
minikube dashboard   
````
Remote-Zugriff erstellen
```
ssh -i <LOCATION TO SSH PRIVATE KEY> -L <LOCAL PORT>:localhost:<REMOTE PORT ON WHICH MINIKUBE DASHBOARD IS RUNNING> user-name@IP
```   
    
Beispiel:
```
ssh -i <keypair>.pem -L 8081:localhost:38923 ubuntu@18.157.79.90
```
Dashboard aufrufen im Web Browser

Beispiel:
```
localhost:8081/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/
```



