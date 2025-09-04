# Team-Sanke
Team Snake 
Setup a private Docker Registry, which has its storage on AWS S3
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
Deploy yeasy/simple-web - a simple web application by using Docker container with 
the private Docker Registry. (https://hub.docker.com/r/yeasy/simple-web/) 

Create config.yml for Registry
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
  
Set up Docker's apt repository.
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

Install the Docker packages.
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

Verify that the installation is successful by running the hello-world image:
sudo docker run hello-world

Verify that the installation is successful by running the hello-world image:
sudo service docker start
sudo docker run hello-world
Run Docker Registry
docker run -d \
  -p 5000:5000 \
  --name registry \
  -v $(pwd)/config.yml:/etc/docker/registry/config.yml \
  registry:2
Deploy yeasy/simple-web Using Private Registry
Pull and Tag the Image
docker pull yeasy/simple-web
docker tag yeasy/simple-web localhost:5000/simple-web
Push to Private Registry
docker push localhost:5000/simple-web
Run the Container
docker run -d --name simple-web -p 8080:80 localhost:5000/simple-web


