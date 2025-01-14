@Library('my-shared-library') _

pipeline {
    agent any

    parameters {
        choice(
            name: 'action', 
            choices: ['create', 'delete'], 
            description: 'Choose create/Destroy'
        )
        string(
            name: 'ImageName', 
            description: "Name of the docker build", 
            defaultValue: 'javapp'
        )
        string(
            name: 'ImageTag', 
            description: "Tag of the docker build", 
            defaultValue: 'v1'
        )
        string(
            name: 'DockerHubUser', 
            description: "Name of the DockerHub user", 
            defaultValue: 'debabratakunty'
        )
    }

    stages {
        stage('Git Checkout') {
            when { 
                expression {  
                    params.action == 'create' 
                } 
            }
            steps {
                gitCheckout(
                    branch: "main",
                    url: "https://github.com/mostwanteddeb/Java_app_3.0.git"
                )
            }
        }
        
        stage('Unit Test Maven') {
            when { 
                expression {  
                    params.action == 'create' 
                } 
            }
            steps {
                script {
                    mvnTest()
                }
            }
        }
        
        stage('Integration Test Maven') {
            when { 
                expression {  
                    params.action == 'create' 
                } 
            }
            steps {
                script {
                    mvnIntegrationTest()
                }
            }
        }
        
        stage('Maven Build') {
            when { 
                expression {  
                    params.action == 'create' 
                } 
            }
            steps {
                script {
                    mvnBuild()
                }
            }
        }
        
        stage('Artifactory push to JFrog') {
            when { 
                expression {  
                    params.action == 'create' 
                } 
            }
            steps {
                script {
                    echo "Pushing to JFrog Artifactory"
                    withCredentials([
                        usernamePassword(
                            credentialsId: "jfrog",
                            usernameVariable: "USER",
                            passwordVariable: "PASS"
                        )
                    ]) {
                        sh "curl -X PUT -u '$USER:$PASS' -T /var/lib/jenkins/workspace/ci_pipe/target/kubernetes-configmap-reload-0.0.1-SNAPSHOT.jar http://44.201.233.51:8082/artifactory/example-repo-local/kubernetes-configmap-reload-0.0.1-SNAPSHOT.jar"
                    }
                }
            }
        }
        
        stage('Docker Image Build') {
            when { 
                expression {  
                    params.action == 'create' 
                } 
            }
            steps {
                script {
                    dockerBuild(params.ImageName, params.ImageTag, params.DockerHubUser)
                }
            }
        }
        
        stage('Docker Image Scan: Trivy') {
            when { 
                expression {  
                    params.action == 'create' 
                } 
            }
            steps {
                script {
                    dockerImageScan(params.ImageName, params.ImageTag, params.DockerHubUser)
                }
            }
        }
        
        stage('Docker Image Push: DockerHub') {
            when { 
                expression {  
                    params.action == 'create' 
                } 
            }
            steps {
                script {
                    dockerImagePush(params.ImageName, params.ImageTag, params.DockerHubUser)
                }
            }
        }
        
        stage('Docker Image Cleanup: DockerHub') {
            when { 
                expression {  
                    params.action == 'create' 
                } 
            }
            steps {
                script {
                    dockerImageCleanup(params.ImageName, params.ImageTag, params.DockerHubUser)
                }
            }
        }
    }
}
