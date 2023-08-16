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
