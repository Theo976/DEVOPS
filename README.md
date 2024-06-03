# DevOps Vassal Théo


# TP1 - Docker


## Directory Organization
The project is organized into several key directories. The sql-scripts/ directory contains all the SQL scripts and configuration files needed for database setup. The simple-api-student-main/ directory houses the source code and Dockerfile necessary for the backend API. For the web front, the http-server/ folder includes the HTML file and corresponding Dockerfile. Additionally, the docker-compose.yml file serves as the central configuration file for managing Docker Compose, facilitating the orchestration of multi-container Docker applications.


## Project Objectives
The primary objectives of this project encompass a variety of DevOps tasks aimed at deploying a robust web application. Initially, the project focuses on configuring and launching a PostgreSQL database inside a Docker container, ensuring the database is set up correctly and is ready to handle requests. Following the database setup, the next goal is to develop and deploy a Java backend API within another Docker container, linking it to the database for data interactions.

In addition to backend services, the project includes setting up a simple HTTP server. This server not only serves web content but is also configured as a reverse proxy, directing incoming requests to the appropriate backend service seamlessly. This setup enhances security and efficiency by abstracting the backend services from direct user access.

Finally, a significant part of the project involves using Docker Compose to manage and orchestrate all the involved containers. Docker Compose allows for defining and running multi-container Docker applications, making it easier to launch, configure, and manage the lifecycle of all project components collectively. This holistic approach ensures that each component is deployed consistently and can interact with others effectively in a controlled environment.



# 1 Database
### 1. Build the PostgreSQL Image

```bash
docker build -t mypostgres:v1.0 ./database
```


### 2. Create the Docker Network

```bash
docker network create app-network
```


Why should we run the container with a flag -e to give the environment variables?

Utilizing the -e flag to provide environment variables when running a container is advisable as it enhances security and configuration flexibility. This approach avoids embedding sensitive information directly into the Dockerfile, ensuring that sensitive data can be managed and modified securely and conveniently outside of the container's build process.

### 3. Run  PostgreSQL Container on the Network

```bash
docker run --name mydatabase -e POSTGRES_DB=db -e POSTGRES_USER=usr -e POSTGRES_PASSWORD=pwd -p 5432:5432 --network app-network -v /my/own/datadir:/var/lib/postgresql/data -d mypostgres:v1.0
```

### 4. Execution script verification 

```bash
docker exec -it my-db psql -U usr -d db
```


### 5. Run Adminer on the Network

```bash
docker run -p "8090:8080" --net=app-network --name=adminer -d adminer
```

To access the PostgreSQL database, navigate to https://localhost:8090 in your web browser and log in using the credentials provided below.

- Server: mydatabase
- User: usr
- Password: pwd
- Database: db





Why do we need a volume to be attached to our postgres container?

Attaching a volume to the PostgreSQL container is essential for data persistence. This setup ensures that the database maintains its data even if the container is stopped or removed.


Why do we need a multistage build? And explain each step of this Dockerfile ?

A multistage build process is crucial for optimizing Docker images. It separates the build environment from the runtime environment, which leads to a final image that is not only smaller and more secure but also more efficient.




# 2. Backend API
### Target java 

```bash
javac Main.java
```


### DockerFile API 


```bash
FROM maven:3.8.6-amazoncorretto-17 AS build
WORKDIR /app
COPY pom.xml /app
RUN mvn dependency:go-offline
COPY src /app/src
RUN mvn package -DskipTests


FROM amazoncorretto:17
WORKDIR /app
COPY --from=build /app/target/*.jar /app/app.jar
ENTRYPOINT ["java", "-jar", "app.jar"]
```


### Build springboot image

```bash
docker build -t springboot-app .
```


### Dockerfile backend_api 


```bash
FROM openjdk:23-slim-bookworm

COPY ./Main.class /usr/src/myapp/Main.class

WORKDIR /usr/src/myapp

CMD ["java", "Main"]
```


### Run the container

```bash
docker run --name simple-api-student -p 8080:8080 --network app-network springboot-app-mai
```





Why do we need a multistage build? And explain each step of this Dockerfile.

Employing a multistage build is essential to enhance the Docker image by distinguishing the build environment from the runtime environment. This distinction helps produce a final image that is more compact, secure, and efficient.



When we navigate to https://localhost:8080/departments/IRC/students, the following information is displayed:


```json
[
  {
    "id": 1,
    "firstname": "Eli",
    "lastname": "Copter",
    "department": {
      "id": 1,
      "name": "IRC"
    }
  }
]
```



# 3. Http server
### Index HTML file 


```html
<!DOCTYPE html>
<html>
<head>
  <title>Welcome</title>
</head>
<body>
  <h1>Hello, welcome to my simple HTTP server!</h1>
</body>
</html>
```



### Pull httpd image 

```bash
docker pull httpd
```


### Build httpd image

```bash
docker build -t my-http-server .
```


### Run container

```bash
docker run -d -p 80:80 --network app-network --name my-web-server my-http-server
```

### Transfer the Configuration to the httpd.conf File Within the Container

```bash
docker cp httpd.conf my-web-server:/usr/local/apache2/conf/httpd.conf
```





Why do we need a reverse proxy?

A reverse proxy is essential for directing client requests to the correct backend service, improving security, facilitating load balancing, and establishing a unified access point for the application.



# 4. Docker Compose configuration file 


```yaml
version: '3.7'
 
services:
    backend:
        build: ./simple-api-student-main
        container_name: "simple_api_student"

        environment:
            DB_host: database
            DB_port: 5432
            DB_name: db
            DB_user: usr
            DB_mdp: pwd

        ports:
          - "8080:8080"
        networks:
          - app-network
        depends_on:
          - database
 
 
    database:
        
        build: ./sql-scripts
        container_name: "database"

        environment:
          POSTGRES_DB: db
          POSTGRES_USER: usr
          POSTGRES_PASSWORD: pwd
        
        ports:
          - "5432:5432"
          
        networks:
          - app-network
 

    httpd:
        
        build: ./http-server
        ports:
          - "80:80"
        networks:
          - app-network
        depends_on:
         - backend
         - database
 
networks:
    app-network:
```

### Run Docker compose file

```bash
docker compose up
```



Why is docker-compose so important?

Docker Compose is valuable because it makes managing multi-container Docker applications easier. It allows you to define and orchestrate all services using just one configuration file, thereby enhancing the efficiency of development, testing, and deployment activities.


# 5. Publish
### Create tag 


```bash
docker tag mypostgres:v1.0 theo976/my-database:lastest
```
```bash
docker tag springboot-app-main:latest theo976/api_backend:v1.0 
```
```bash
docker tag my-http-server:latest theo976/my-http-server:v1.0
```



### Push on DockerHub 

```bash
docker push theo976/my-database:lastest
```

```bash
docker push theo976/api_backend:v1.0
```

```bash
docker push theo976/my-http-server:v1.0
```


Why do we put our images into an online repo?

Uploading our images to an online repository facilitates the sharing, versioning, and deployment of these images across various environments and among team members. This ensures that builds are consistent and can be reproduced reliably.






# TP2 - Github Actions
### Build and test your Application


```bash
mvn clean verify --file simple-api-student-main/pom.xml
```


```yaml
name: CI devops 2024

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  test-backend:
    runs-on: ubuntu-22.04

    steps:
      - name: Checkout code
        uses: actions/checkout@v2.5.0

      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'temurin'

      - name: Build and test with Maven
        run: mvn -B verify sonar:sonar -Dsonar.projectKey=devopstheo2_devops -Dsonar.organization=devopstheo2 -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=${{ secrets.SONAR_TOKEN }} --file simple-api-student-main/pom.xml

  build-and-push-docker-image:
    needs: test-backend
    runs-on: ubuntu-22.04
    steps:
      - name: Checkout code
        uses: actions/checkout@v2.5.0

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v1

      - name: Login to DockerHub
        uses: docker/login-action@v1
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build image and push backend
        uses: docker/build-push-action@v3
        with:
          context: ./simple-api-student-main
          tags: ${{secrets.DOCKERHUB_USERNAME}}/api_backend:latest
          push: ${{ github.ref == 'refs/heads/main' }}

      - name: Build image and push database
        uses: docker/build-push-action@v3
        with:
          context: ./sql-scripts
          tags: ${{secrets.DOCKERHUB_USERNAME}}/my-database:latest
          push: ${{ github.ref == 'refs/heads/main' }}

      - name: Build image and push httpd
        uses: docker/build-push-action@v3
        with:
          context: ./http-server
          tags: ${{secrets.DOCKERHUB_USERNAME}}/my-http-server:latest
          push: ${{ github.ref == 'refs/heads/main' }}

```





This GitHub Actions workflow is designed to trigger on pushes or pull requests to the main and develop branches. It comprises two main jobs: the test-backend, which tests the backend using Maven and SonarCloud, and the build-and-push-docker-image, which handles the construction and distribution of Docker images. The test-backend job begins by checking out the code, setting up JDK 17, and using Maven to both verify the code and conduct a SonarCloud analysis. Following this, the build-and-push-docker-image job, which is contingent on the successful completion of the test-backend job, checks out the code once more, logs into DockerHub, and proceeds to build and push Docker images for the backend, database, and HTTP server, provided the action was triggered by changes to the main branch.



How to Add the SonarCloud Token to Repository Secrets


To set up a new secret for your repository, follow these steps:

Access the settings of your repository.
Locate and click on "Secrets" or "Variables," depending on the configuration options available on your CI platform.
Create a new secret with the name SONAR_TOKEN.
Enter your SonarCloud token as the value for this secret.



# TP3 - Ansible
### Install Ansible

```bash
brew install ansible
```



### Check Ansible version

```bash
ansible --version
```



### Set Permissions for SSH Key

```bash
chmod 400 id_rsa
```


### Connect to the host

```bash
ssh -i id_rsa centos@theo.vassal.takima.cloud
```


### Install vim in the host

```bash
sudo yum install vim -y
```



### Run setup file with ssh key

```bash
ansible -i ansible/inventory/setup.yml all -m ping --private-key=id_rsa -u centos
```

after that we receive success with Pong response



### Installing Apache via Ansible

```bash
ansible -i ansible/inventory/setup.yml all -m yum -a "name=httpd state=present" --private-key=id_rsa -u centos --become
```




### Updating the Home Page with Ansible

```bash
ansible prod -i ansible/inventory/setup.yml -m shell -a 'echo "<html><h1>Hello World</h1></html>" >> /var/www/html/index.html' --become
```


### Start Apache via Ansible

```bash
ansible all -i ansible/inventory/setup.yml  -m service -a "name=httpd state=started" --become 
```


go on https://theo.vassal.takima.cloud and see "HELLO WORLD"





### Get distribution details via Ansible

```bash
ansible all -i ansible/inventory/setup.yml -m setup -a "filter=ansible_distribution*”
```




### Remove Apache via Ansible

```bash
ansible all -i ansible/inventory/setup.yml -m yum -a "name=httpd state=absent" --become
```



### Start Ansible playbook

```bash
ansible-playbook -i ansible/inventory/setup.yml ansible/playbook.yml
```



### Final Ansible playbook

```yaml
- hosts: all
  gather_facts: false
  become: true




  - name: Install device-mapper-persistent-data
    yum:
      name: device-mapper-persistent-data
      state: latest

  - name: Install lvm2
    yum:
      name: lvm2
      state: latest

  - name: add repo docker
    command:
      cmd: sudo yum-config-manager --add-repo=https://download.docker.com/linux/centos/docker-repo

  - name: Install Docker
    yum:
      name: docker-ce
      state: present

  - name: Install python3
    yum:
      name: python3
      state: present

  - name: Install docker with Python 3
    pip:
      name: docker
      executable: pip3
    vars:
      ansible_python_interpreter: /usr/bin/python3

  - name: Make sure Docker is running
    service: name=docker state=started
    tags: docker



# App Builder using the Docker role
# - hosts: all
#   become: true
#   roles: 
#     - docker


- hosts: all
  become: true
  vars:
    ansible_python_interpreter: /usr/bin/python

  roles:
    - install_docker
    - create_network
    - launch_database
    - launch_app
    - launch_proxy

```


### Playbook  

The playbook is designed to operate across all hosts, escalating privileges with the become command while opting not to gather facts. It begins by installing essential packages required for Docker functionality, such as device-mapper-persistent-data and lvm2, and proceeds to add the Docker repository. Following this setup, it installs Docker and Python 3, along with the Docker Python module via pip3. The playbook ensures the Docker service is active and running. Subsequently, it utilizes various roles to execute multiple tasks: installing Docker, creating a network infrastructure, and deploying the database, application, and proxy services effectively.




### Creating a Role for Docker Installation

```bash 
ansible-galaxy init roles/docker
```



### install_docker role, main file

```yaml
---
- name: Install required packages
  yum:
    name:
      - yum-utils
      - device-mapper-persistent-data
      - lvm2
    state: present

- name: Add Docker repository
  command: yum-config-manager --add-repo=https://download.docker.com/linux/centos/docker-repo

- name: Install Docker
  yum:
    name: docker-ce
    state: present

- name: Start Docker service
  service:
    name: docker
    state: started
    enabled: yes
```




### create_network role, main file

```yaml
---
- name: Create Docker network
  docker_network:
    name: app-network
```




### lauch_database role, main file

```yaml
---
- name: Launch database container
  docker_container:
    name: database
    image: theo976/my-database:latest
    ports:
      - "5432:5432"
    networks:
      - name: app-network
```






### lauch_app role, main file

```yaml
---
- name: Launch backend container
  docker_container:
    name: simple_api_student
    image: theo976/api_backend:latest
    ports:
      - "8080:8080"
    networks:
      - name: app-network
```






### lauch_proxy role, main file

```yaml
---
- name: Launch httpd container
  docker_container:
    name: httpd
    image: theo976/my-http-server:latest
    ports:
      - "80:80"
    networks:
      - name: app-network
```


### Deploy all the application 

```bash
ansible-playbook -i ansible/inventory/setup.yml ansible/playbook.yml
```




### Workflow deploy.yml for deployment in continue 


```yaml
name: Deploy Application

on:
  workflow_run:
    workflows: ["CI devops 2024"]
    types:
      - completed

jobs:
  deploy:
    if: ${{ github.event.workflow_run.conclusion == 'success' }}
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v2

    - name: Set up SSH
      uses: webfactory/ssh-agent@v0.5.3
      with:
        ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}

    - name: Install Ansible
      run: |
        sudo apt-get update
        sudo apt-get install -y ansible

    - name: Disable host key checking
      run: echo "ANSIBLE_HOST_KEY_CHECKING=False" >> $GITHUB_ENV

    - name: Run deployment playbook
      run: |
        ansible-playbook -i ansible/inventory/setup.yml ansible/playbook.yml
      env:
        ANSIBLE_HOST_KEY_CHECKING: False

```



### Workflow deploy explaination 

This workflow activates upon the successful completion of the "CI devops 2024" workflow.
It includes a job named "deploy" that operates on the latest Ubuntu environment.
This job first checks out the repository and configures SSH with a private key secured in GitHub Secrets.
It then proceeds to install Ansible on the runner and disables host key checking for smoother operations.
The final step involves executing an Ansible playbook that deploys the application.





## Load Balancing BONUS


### install_grafana role, main file

```yaml
---
- name: Add Grafana repository
  yum_repository:
    name: grafana
    description: Grafana Repository
    baseurl: https://packages.grafana.com/oss/rpm
    gpgcheck: yes
    gpgkey: https://packages.grafana.com/gpg.key
    enabled: yes

- name: Install Grafana
  yum:
    name: grafana
    state: present

- name: Start and enable Grafana
  service:
    name: grafana-server
    state: started
    enabled: yes
```





### lauch_backends role, main file

```yaml
---
- name: Launch backend instance 1
  docker_container:
    name: backend-1
    image: theo976/api_backend:latest
    ports:
      - "8081:8080"
    networks:
      - name: app-network

- name: Launch backend instance 2
  docker_container:
    name: backend-2
    image: theo976/api_backend:latest
    ports:
      - "8082:8080"
    networks:
      - name: app-network
```




### lauch_proxy role, main file

```yaml
---
- name: Launch httpd container
  docker_container:
    name: httpd
    image: theo976/my-http-server:latest
    ports:
      - "80:80"
    networks:
      - name: app-network

```




### httpd.conf file modification with 2 backends

```bash

<VirtualHost *:80>

ServerName theo.vassal.takima.cloud


ProxyPreserveHost On

<Proxy balancer://mycluster>
        BalancerMember http://backend-1:8081
        BalancerMember http://backend-2:8082
        ProxySet lbmethod=byrequests
</Proxy>


    ProxyPass / balancer://mycluster/
    ProxyPassReverse / balancer://mycluster/

#ProxyPass / http://simple_api_student_main:8080/
#ProxyPassReverse / http://simple_api_student_main:8080/

</VirtualHost>

LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_http_module modules/mod_proxy_http.so
LoadModule proxy_balancer_module modules/mod_proxy_balancer.so
LoadModule lbmethod_byrequests_module modules/mod_lbmethod_byrequests.so

```