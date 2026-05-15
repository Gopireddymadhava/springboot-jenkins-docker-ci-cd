Here’s the cleaned and properly structured version of your setup steps for Jenkins, SonarQube, Docker, and SSH integration between two EC2 instances on Amazon Web Services.

Jenkins EC2 Setup (Amazon Linux)
1. Add Jenkins Repository
sudo wget -O /etc/yum.repos.d/jenkins.repo \
https://pkg.jenkins.io/redhat-stable/jenkins.repo
2. Import Jenkins GPG Key
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
3. Upgrade Packages
sudo dnf upgrade -y
4. Install Jenkins
sudo dnf install jenkins -y
5. Install Java 21
sudo dnf install -y java-21-amazon-corretto

Verify Java installation:

java -version
6. Start Jenkins Service
sudo systemctl enable jenkins
sudo systemctl start jenkins

Check Jenkins status:

sudo systemctl status jenkins
SonarQube Setup Using Docker (on Jenkins EC2)

Using Docker is the recommended and simplest approach for Amazon Linux.

Components:

Docker
SonarQube container
PostgreSQL container

Ports:

SonarQube → 9000
PostgreSQL → internal only
1. Install Docker
sudo dnf install docker -y

Start Docker:

sudo systemctl enable docker
sudo systemctl start docker

Verify installation:

docker --version
2. Give EC2 User Docker Access
sudo usermod -aG docker ec2-user

Apply group changes:

newgrp docker
3. Create Docker Network
docker network create sonarnet
4. Run PostgreSQL Container
docker run -d \
  --name sonarqube-db \
  --network sonarnet \
  -e POSTGRES_USER=sonar \
  -e POSTGRES_PASSWORD=sonar123 \
  -e POSTGRES_DB=sonarqube \
  postgres:15
5. Run SonarQube Container
docker run -d \
  --name sonarqube \
  --network sonarnet \
  -p 9000:9000 \
  -e SONAR_JDBC_URL=jdbc:postgresql://sonarqube-db:5432/sonarqube \
  -e SONAR_JDBC_USERNAME=sonar \
  -e SONAR_JDBC_PASSWORD=sonar123 \
  sonarqube:lts-community
6. Open Security Group Port

In AWS EC2 Security Group, add:

Type	Port	Source
Custom TCP	9000	0.0.0.0/0
7. Access SonarQube

Open browser:

http://YOUR_PUBLIC_IP:9000
8. Default SonarQube Credentials

Username:

admin

Password:

admin

It will prompt you to change the password after login.

9. Verify Running Containers
docker ps

You should see:

sonarqube
sonarqube-db
If SonarQube Fails to Start

Most common reason:

missing Linux kernel settings
insufficient memory

Run these commands on Jenkins EC2.

1. Set Required Kernel Parameters
sudo sysctl -w vm.max_map_count=524288
sudo sysctl -w fs.file-max=131072
2. Make Settings Permanent
echo "vm.max_map_count=524288" | sudo tee -a /etc/sysctl.conf

echo "fs.file-max=131072" | sudo tee -a /etc/sysctl.conf

sudo sysctl -p
3. Restart Containers
docker start sonarqube-db
docker start sonarqube
4. Verify Status
docker ps

Expected:

sonarqube       Up
sonarqube-db    Up
5. Check Logs If Still Failing
docker logs sonarqube
Configure SSH Access Between Jenkins EC2 and Docker EC2

These commands should be executed on the Jenkins EC2 instance.

1. Generate SSH Key
ssh-keygen

Press Enter for all prompts.

2. Copy Public Key to Docker EC2

If ssh-copy-id is available:

ssh-copy-id ec2-user@<DOCKER_EC2_PRIVATE_IP>
3. If ssh-copy-id Is Not Available

Display public key:

cat ~/.ssh/id_rsa.pub

Copy the complete output.

Commands to Run on Docker EC2

Create SSH directory:

mkdir -p ~/.ssh

Open authorized_keys file:

nano ~/.ssh/authorized_keys

Paste the copied public key from Jenkins EC2.

Save and exit:

CTRL + O
Enter
CTRL + X
Set Proper Permissions
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
Test SSH Connectivity

Run this command from Jenkins EC2:

ssh ec2-user@<DOCKER_EC2_PRIVATE_IP>

If configured correctly, it should log in without asking for a password.
