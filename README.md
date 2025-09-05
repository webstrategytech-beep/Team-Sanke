1.Team Snake is working on a new project that serves payment services globally. They have the 
following services to be deployed: 
a) Web server 
b) External API Service 
c) Internal API Service 
d) Payment Processor Integration Service 
e) Redis Cache 
f) Relational Database 
Please provide a network architecture diagram, as well as suitable solutions that considers 
Security, High Availability / Reliability, Scalability, Performance and Observability

 


1.Setup a private Docker Registry, which has its storage on AWS S3

{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Statement1",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "S3:GetObject",
            "Resource": "arn:aws:s3:::teamsnake/*"
        }
    ]
}
2.Deploy yeasy/simple-web - a simple web application by using Docker container with 
the private Docker Registry. (https://hub.docker.com/r/yeasy/simple-web/) 

•	Create config.yml for Registry
version: 0.1
storage:
  s3:
    accesskey: <AWS_ACCESS_KEY>
    secretkey: <AWS_SECRET_KEY>
    region: us-east-1
    bucket: my-docker-registry
    encrypt: true
    secure: true
    v4auth: true
    chunksize: 5242880
    rootdirectory: /registry
http:
  addr: :5000
  
a.Set up Docker's apt repository.
b.Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

c. Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

d.Install the Docker packages.
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

e. Verify that the installation is successful by running the hello-world image:
sudo docker run hello-world

f. Verify that the installation is successful by running the hello-world image:
sudo service docker start
sudo docker run hello-world
g.Run Docker Registry
docker run -d \
  -p 5000:5000 \
  --name registry \
  -v $(pwd)/config.yml:/etc/docker/registry/config.yml \
  registry:2
h.Deploy yeasy/simple-web Using Private Registry
i.Pull and Tag the Image
docker pull yeasy/simple-web
docker tag yeasy/simple-web localhost:5000/simple-web
j.Push to Private Registry
docker push localhost:5000/simple-web
k.Run the Container
docker run -d --name simple-web -p 8080:80 localhost:5000/simple-web
3. Setup a reverse-proxy using Nginx infront of one yeasy/simple-web container

events {}
http {
  server {
    listen 80;
    location / {
      proxy_pass http://simple-web:80;
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
    }
  }
}
a.Docker Network
docker network create webnet
b.Run Container

docker run -d --name simple-web --network webnet localhost:5000/simple-web
docker run -d --name nginx-proxy --network webnet -p 80:80 \
  -v $(pwd)/nginx.conf:/etc/nginx/nginx.conf nginx

4. [Bonus] Setup a load balancer using Nginx with yeasy/simple-web as its web 
Application

a. Run Multiple App Containers
docker run -d --name simple-web1 --network webnet localhost:5000/simple-web
docker run -d --name simple-web2 --network webnet localhost:5000/simple-web

b. Update nginx.conf for Load Balancing
http {
  upstream simple_web_cluster {
    server simple-web1:80;
    server simple-web2:80;
  }

  server {
    listen 80;
    location / {
      proxy_pass http://simple_web_cluster;
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
    }
  }
}

c. Restart Nginx Container
docker rm -f nginx-proxy
docker run -d --name nginx-proxy --network webnet -p 80:80 \
  -v $(pwd)/nginx.conf:/etc/nginx/nginx.conf nginx
 
 
 
 

 
 

 

 
 
 

 



