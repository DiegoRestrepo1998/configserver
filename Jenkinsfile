pipeline {
    agent any
    environment {
        PROJECT_ID = 'My First Project'
                CLUSTER_NAME = 'cluster-jenkins-project'
                LOCATION = 'us-central1'
                CREDENTIALS_ID = 'JenkinsProject'
    }
    
    stages {
        stage('Check Git...') {
            steps{
                checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/DiegoRestrepo1998/configserver.git']])
            }
        }
        stage('Building image...') {
            steps {
                script {
                    app = docker.build("dfrestrepo1998/microservicio_configserver:1.0.0")
                    }
            }
        }
        
        stage('Pushing image...') {
            environment {
                def imageName = "dfrestrepo1998/microservicio_configserver:1.0.0"
            }
            steps {
                script {
                    withCredentials([string(credentialsId: 'dockerhub', variable: 'dockerhub')]) {
                        sh 'docker login -u dfrestrepo1998 -p ${dockerhub}'
                        sh "docker push ${imageName}"
                    }
                }
            }
        }
    
        stage('Deploying to K8s') {
            steps{
                echo "Deployment started ..."
                sh 'ls -ltr'
                sh 'pwd'
                sh "sed -i 's/pipeline:latest/pipeline:1.0.0/g' configserver_deploy.yml"
                step([$class: 'KubernetesEngineBuilder', \
                  projectId: env.PROJECT_ID, \
                  clusterName: env.CLUSTER_NAME, \
                  location: env.LOCATION, \
                  manifestPattern: 'configserver_deploy.yml', \
                  credentialsId: env.CREDENTIALS_ID, \
                  verifyDeployments: true])

                step([$class: 'KubernetesEngineBuilder', \
                  projectId: env.PROJECT_ID, \
                  clusterName: env.CLUSTER_NAME, \
                  location: env.LOCATION, \
                  manifestPattern: 'configmap.yml', \
                  credentialsId: env.CREDENTIALS_ID, \
                  verifyDeployments: true])
                }
            }
        }    
}