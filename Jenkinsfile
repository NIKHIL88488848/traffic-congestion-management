pipeline {
    agent any
    
    parameters {
        string(name: 'DOCKER_TAG', defaultValue: 'latest', description: 'Docker image tag version label')
        choice(name: 'DEPLOY_ENV', choices: ['Staging', 'Production'], description: 'Deployment cluster environment target')
    }

    environment {
        HUB_REPO = 'NIKHIL8848848/traffic-congestion-management'
        CRED_ID  = 'docker-hub-credentials'
    }

    stages {
        stage('Pull & Workspace Init') {
            steps {
                echo "Pulling application source files for context environment: ${params.DEPLOY_ENV}..."
                checkout scm
            }
        }

        stage('Build & Test Verification') {
            parallel {
                stage('Docker Core Build') {
                    steps {
                        echo "Building container asset tags..."
                        sh "docker build -t ${HUB_REPO}:${params.DOCKER_TAG} ."
                    }
                }
                stage('Health Pre-Check') {
                    steps {
                        echo "Running asset structure assertions..."
                        sh "grep -q 'Traffic Management' index.html"
                    }
                }
            }
        }

        stage('Registry Distribution') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${CRED_ID}", passwordVariable: 'HUB_PASS', usernameVariable: 'HUB_USER')]) {
                    sh "echo \$HUB_PASS | docker login -u \$HUB_USER --password-stdin"
                    sh "docker push ${HUB_REPO}:${params.DOCKER_TAG}"
                }
            }
        }

        stage('Kubernetes Cluster Deployment') {
            steps {
                echo "Mutating deployment targets to point to tag: ${params.DOCKER_TAG}"
                sh "sed -i 'image:.*traffic-congestion-management:.*' deployment.yaml"
                sh "sed -i 'image: ${HUB_REPO}:${params.DOCKER_TAG}' deployment.yaml"
                
                echo "Applying specifications onto local cluster infrastructure..."
                sh "kubectl apply -f deployment.yaml"
            }
        }
    }
}
