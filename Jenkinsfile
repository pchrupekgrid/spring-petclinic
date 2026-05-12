pipeline {
    agent any
    
    tools {
        // Ta nazwa MUSI być taka sama jak ta, którą wpiszesz w ustawieniach Tools
        dockerTool 'my-docker' 
    }

    environment {
        // Połączenie z Twoim drugim kontenerem (DinD)
        DOCKER_HOST = 'tcp://jenkins-docker:2376'
        DOCKER_TLS_VERIFY = '1'
        DOCKER_CERT_PATH = '/certs/client'
    }

    stages {
        stage('pipeline_a') {
            when { not { branch 'main' } }
            stages {
                stage('Checkstyle_a') {
                    steps {
                        sh './gradlew checkstyleMain'
                    }
                    post {
                        always { 
                            archiveArtifacts artifacts: 'build/reports/checkstyle/*.html'
                        }
                    }
                }
                stage('Test_a') {
                    steps {
                        sh './gradlew test'
                    }
                }
                stage('Build_a_and_push') {
                    steps {
                        sh './gradlew bootJar -x test'
                        script {
                            env.GIT_COMMIT_SHORT = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                            docker.withRegistry('http://nexus:8083', 'nexus-creds') {
                                def image = docker.build("spring-petclinic:${env.GIT_COMMIT_SHORT}")
                                image.push()
                            }
                        }
                    }
                }
            }
        }

        stage('Build_b_and_push') {
            when { branch 'main' }
            steps {
                script {
                    docker.withRegistry('http://nexus:8082', 'nexus-creds') {
                        def image = docker.build("spring-petclinic:latest")
                        image.push()
                    }
                }
            }
        }
    }
}