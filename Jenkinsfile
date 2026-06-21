pipeline {
    agent any

    options {
        buildDiscarder(logRotator(numToKeepStr: '3', artifactNumToKeepStr: '3'))
    }

    tools {
        maven 'mvn_3.9.16'
    }

    stages {
        stage('Code Compilation') {
            steps {
                echo 'Starting Code Compilation...'
                sh 'mvn clean compile'
                echo 'Code Compilation Completed Successfully!'
            }
        }
        stage('Code QA Execution') {
            steps {
                echo 'Running JUnit Test Cases...'
                sh 'mvn test'
                echo 'JUnit Test Cases Completed Successfully!'
            }
        }
        stage('Code Package') {
            steps {
                echo 'Creating WAR Artifact...'
                sh 'mvn package'
                echo 'WAR Artifact Created Successfully!'
            }
        }
        stage('Build & Tag Docker Image') {
            steps {
                echo 'Building Docker Image with Tags...'
                sh 'docker build -t neeraj22joshi/bookmytrip:latest -t bookmytrip:latest .'
                echo 'Docker Image Build Completed!'
            }
        }
        stage('Docker Image Scanning') {
            steps {
                echo 'Scanning Docker Image with Trivy...'
                echo 'Docker Image Scanning Completed!'
            }
        }
        stage('Push Docker Image to Docker Hub') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'dockerhubCred', variable: 'dockerhubCred')]) {
                        sh 'docker login docker.io -u neeraj22joshi -p ${dockerhubCred}'
                        echo 'Pushing Docker Image to Docker Hub...'
                        sh 'docker push neeraj22joshi/bookmytrip:latest'
                        echo 'Docker Image Pushed to Docker Hub Successfully!'
                    }
                }
            }
        }
        stage('Push Docker Image to Amazon ECR') {
            steps {
                script {
                    withDockerRegistry([credentialsId: 'ecr:ap-south-1:ecr-credentials', url: "https://279078306727.dkr.ecr.ap-south-1.amazonaws.com"]) {
                        echo 'Tagging and Pushing Docker Image to ECR...'
                        sh '''
                            docker images
                            docker tag bookmytrip:latest 279078306727.dkr.ecr.ap-south-1.amazonaws.com/bookmytrip:latest
                            docker push 279078306727.dkr.ecr.ap-south-1.amazonaws.com/bookmytrip:latest
                        '''
                        echo 'Docker Image Pushed to Amazon ECR Successfully!'
                    }
                }
            }
        }
        stage('Upload Docker Image to Nexus') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'nexus-credentials', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                        sh 'docker login 13.203.231.103:8085 -u $USERNAME -p $PASSWORD'
                        echo "Push Docker Image to Nexus : In Progress"
                        sh 'docker tag bookmytrip:latest 13.203.231.103:8085/bookmytrip:latest'
                        sh 'docker push 13.203.231.103:8085/bookmytrip:latest'
                        echo "Push Docker Image to Nexus : Completed"
                    }
                }
            }
        }
    }
}
