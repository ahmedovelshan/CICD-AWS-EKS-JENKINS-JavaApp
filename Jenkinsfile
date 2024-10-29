pipeline {
    agent any
    tools {
        jdk 'jdk'
        maven 'maven'
    }
    parameters {
        string(name: 'AWS_ACCESS_KEY_ID', description: 'Enter AWS Access Key ID')
        string(name: 'AWS_SECRET_ACCESS_KEY', description: 'Enter AWS Secret Access Key')
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        AWS_ACCESS_KEY_ID = "${params.AWS_ACCESS_KEY_ID}"
        AWS_SECRET_ACCESS_KEY = "${params.AWS_SECRET_ACCESS_KEY}"
        AWS_DEFAULT_REGION = "eu-central-1"
    }
    
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/ahmedovelshan/full-stack-blogging-app.git'
            }
        }
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        stage('Trivy FS') {
            steps {
                sh "trivy fs . --format table -o fs.html"
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqubeServer') {
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=Blogging-app \
                        -Dsonar.projectKey=Blogging-app \
                        -Dsonar.java.binaries=target
                    '''
                }
            }
        }
        stage('Build') {
            steps {
                sh "mvn package"
            }
        }
        stage('Publish Artifacts') {
            steps {
                withMaven(globalMavenSettingsConfig: 'maven-settings') {
                    sh "mvn deploy"
                }
            }
        }
        stage('Docker Build & Tag') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dockerhub-cred', url: 'https://index.docker.io/v1/') {
                        sh "docker build -t ahmedovelshan/gab-blogging-app2 ."
                    }
                }
            }
        }
        stage('Trivy Image Scan') {
            steps {
                sh "trivy image --format table -o image.html ahmedovelshan/gab-blogging-app2:latest"
            }
        }
        stage('Docker Push Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dockerhub-cred', url: 'https://index.docker.io/v1/') {
                        sh "docker push ahmedovelshan/gab-blogging-app2"
                    }
                }
            }
        }
        stage("Deploy to EKS") {
            steps {
                script {
                    sh "aws eks update-kubeconfig --region eu-central-1 --name Blue-green-eks-cluster"
                    sh "kubectl apply -f deployment-service.yml"
                }
            }
        }
    }
}

