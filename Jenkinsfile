@Library('my-shared-library') _

pipeline {
    agent any
    parameters {
        choice(name: 'action', choices: 'create\nrollback', description: 'Create/rollback of the deployment')
        string(name: 'ImageName', description: "Name of the docker build", defaultValue: "kubernetes-configmap-reload")
        string(name: 'ImageTag', description: "Name of the docker build", defaultValue: "v1")
        string(name: 'AppName', description: "Name of the Application", defaultValue: "kubernetes-configmap-reload")
        string(name: 'docker_repo', description: "Name of docker repository", defaultValue: "debarghya499")
    }

    stages {
        stage('Git checkout') {
            when {
                expression { params.action == "create" }
            }
            steps {
                gitCheckout(
                    branch: "main",
                    url: "https://github.com/debarghyasarma/Java_app_3.0.git"
                )
            }
        }

        stage('Unit test Maven') {
            when {
                expression { params.action == "create" }
            }
            steps {
                script {
                    mvnTest()
                }
            }
        }

        stage('Integration test Maven') {
            when {
                expression { params.action == "create" }
            }
            steps {
                script {
                    mvnIntegrationTest()
                }
            }
        }

        stage('Static code analysis') {
            when {
                expression { params.action == "create" }
            }
            steps {
                script {
                    def SonarQubecredentialsId = 'sonarqube-api'
                    statiCodeAnalysis(SonarQubecredentialsId)
                }
            }
        }

        stage('Quality Gate Status Check : Sonarqube') {
            when {
                expression { params.action == "create" }
            }
            steps {
                script {
                    def SonarQubecredentialsId = "sonar-api"
                    QualityGateStatus(SonarQubecredentialsId)
                }
            }
        }

        stage('Build Maven') {
            when {
                expression { params.action == "create" }
            }
            steps {
                script {
                    mvnBuild()
                }
            }
        }

        stage('Docker Image build') {
            when {
                expression { params.action == "create" }
            }
            steps {
                script {
                    dockerBuild("${params.ImageName}", "${params.ImageTag}", "${params.docker_repo}")
                }
            }
        }

        stage('Image Scan Trivy') {
            when {
                expression { params.action == "create" }
            }
            steps {
                script {
                    dockerImageScan("${params.ImageName}", "${params.ImageTag}", "${params.docker_repo}")
                }
            }
        }

        stage('Docker image push') {
            when {
                expression { params.action == "create" }
            }
            steps {
                script {
                    dockerImagePush("${params.ImageName}", "${params.ImageTag}", "${params.docker_repo}")
                }
            }
        }

        stage('Create deployment') {
            when {
                expression { params.action == "create" }
            }
             steps {
                script {
                    sh 'sudo kubectl apply -f ${WORKSPACE}/deployment.yaml' // Point this to your Kubernetes YAML file           
                }
            }
        }

        stage('Pods Deployment') {
            when {
                expression { params.action == "create" }
            }
            steps {
                sh 'sleep 300'
            }
        }

        stage('Rollback deployment') {
            when {
                expression { params.action == "rollback" }
            }
            steps {
                sh """
                    sudo kubectl delete deploy ${params.AppName}
                    sudo kubectl delete svc ${params.AppName}
                """
            }
        }
    }
}
