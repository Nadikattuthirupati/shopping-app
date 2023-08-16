# Containerizing the Shopping-Cart Application with Docker&Kubernets

## Environment variable updates

- Edit src/application.properties
- replace localhost with the container name of mysql db container

```
db.connectionString = jdbc:mysql://mysql:3306/shopping-cart
```

## Creating dockerfiles

- Create dockerfile.application for building application image

```bash
FROM tomcat:9
ADD target/shopping-cart-0.0.1-SNAPSHOT.war /usr/local/tomcat/webapps/shopping-cart-0.0.1-SNAPSHOT.war
EXPOSE 8080
CMD ["catalina.sh", "run"]

```

- Create docerfile.db for building mysql image

```bash
FROM mysql:latest
ADD databases/mysql_query.sql docker-entrypoint-initdb.d/mysql_query.sql
ENV MYSQL_ROOT_PASSWORD root
EXPOSE 3306

```

> Both the images have been built and tested locally first to ensure there is no error and pushed to the repo.

- Create a Jenkinsfile

```bash
pipeline {
    agent any
    tools{
        maven 'jenkins-maven'
    }
    stages{
        stage('Build Maven'){
            steps{
                checkout scm
                sh 'mvn clean install'
            }
        }
        stage('Build docker image'){
            steps{
                script{
                    sh 'docker build -t ShoppingApp/shopping-application -f Dockerfile.application .'
                    sh 'docker build -t ShoppingApp/shopping-db -f Dockerfile.db .'
                }
            }
        }
        stage('Push image to Hub'){
            steps{
                script{
                   withCredentials([string(credentialsId: 'hub-password', variable: 'hub-password')]) {
                   sh 'docker login -u tirupati -p ${hub-password}'

}
                   sh 'docker push ShoppingApp/shopping-application'
                   sh 'docker push ShoppingApp/shopping-db'

                   
                }
            }
        }

    }
}

```

## Jenkins Setup

```bash
docker run -p 8080:8080 -p 50000:50000 --restart=on-failure -d -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts-jdk11

```

- Mount volume of jenkins home "jenkins_home" is to retain data even if container gets deleted

- Browse [http://localhost:8080/](http://localhost:8080/) and complete initial jenkins setup


- In the Dashboard section create a new item as pipeline

- Add the github url of project repo in Source Code Management section of configure tab

- Add pipeline script with scm checkout maven build, docker package

- Create your dockerhub password as "hub-password" credential before building and published to dockerhub repo

##Output

- Exposed 8081 port since 8080 is already being used by jenkins by default

- Same images can be deployed in kubernetes with Kind deployment or statefulset

```
        stage('Deploy to kubernetes'){
            steps{
                script{
                    kubernetesDeploy (configs: 'deploymentservice.yaml', kubeconfigId: 'k8sconfigpwd')
                }
            }
        }
```

## Author

- Tirupati N
- Mail - **nadikattu1999@gmail.com**
