pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Client Build') {
            agent {
                docker {
                    image 'node:10.19.0'
                    args '-u root:root' 
                }   
            }
            steps {
                dir('notes-ui') {
                    sh 'npm install'
                    // sh 'npm install react react-dom react-router-dom react-scripts web-vitals'
                }
            }
        }
        
        stage('Server Build and Test') {
            agent {
                docker {
                    image 'node:10.19.0'
                    args '-u root:root' 
                }   
            }
            steps {
                dir('server') {
                    sh 'npm install'
                    sh 'npm install body-parser cross-env express mocha mysql2 nodemon sequelize should sqlite3 supertest'
                    // sh 'npm test'
                }
            }
        }
        stage('Run linting tests'){
            parallel{
                stage('Lint UI Dockerfile') {
                    steps {
                        dir('notes-ui') {
                            script {
                                // Run Hadolint to lint Dockerfile and save results to hadolint.txt
                                sh 'hadolint Dockerfile > hadolint.txt || true'
                            
                                // Check if hadolint.txt contains any errors
                                def hadolintOutput = readFile('hadolint.txt').trim()
                                if (hadolintOutput) {
                                    error "Hadolint detected errors in Dockerfile:\n${hadolintOutput}"
                                }
                            }
                        }
                    }
                } // Added missing bracket
                stage('Lint server Dockerfile') {
                    steps {
                        dir('server') {
                            script {
                                // Run Hadolint to lint Dockerfile and save results to hadolint.txt
                                sh 'hadolint Dockerfile > hadolint.txt || true'
                            
                                // Check if hadolint.txt contains any errors
                                def hadolintOutput = readFile('hadolint.txt').trim()
                                if (hadolintOutput) {
                                    error "Hadolint detected errors in Dockerfile:\n${hadolintOutput}"
                                }
                            }
                        }
                    }
                }
            } // Closed parallel block
        }
        
        stage('Build Images') {
            steps {
                script {
                    // Set the tag number to the build number
                    def clientTag = "animesh0406/todo-ui:client-${env.BUILD_NUMBER}"
                    def serverTag = "animesh0406/todo-back:server-${env.BUILD_NUMBER}"
        
                    // Build Docker images with the updated tags
                    sh "docker build -t ${clientTag} notes-ui"
                    sh "docker build -t ${serverTag} server"
                }
            }
        }
        
        stage('Push Images to DockerHub') {
            steps {
                script {
                    def clientTag = "animesh0406/todo-ui:client-${env.BUILD_NUMBER}"
                    def serverTag = "animesh0406/todo-back:server-${env.BUILD_NUMBER}"
        
                    withCredentials([usernamePassword(credentialsId: 'your-credentials-id', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                        sh "docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD"
                        sh "docker push ${clientTag}"
                        sh "docker push ${serverTag}"
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                sshagent (credentials: ['your-ssh-credentials-id']){
                    sh "scp -o StrictHostKeyChecking=no -r docker-compose.yml root@192.168.70.80:/root/"
                    sh "ssh -o StrictHostKeyChecking=no -T root@192.168.70.80 'docker-compose up -d'"
                }
            }
        }
    }
}
