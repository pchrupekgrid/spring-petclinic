pipeline {
    agent any

    environment {
        // Łączymy się z Twoim kontenerem DinD przez sieć Dockerową
        // Używamy portu 2375 (z Twojego docker ps), który zazwyczaj nie wymaga TLS
        DOCKER_HOST = 'tcp://jenkins-docker:2375'
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
                            
                            withCredentials([usernamePassword(credentialsId: 'nexus-creds', passwordVariable: 'NEXUS_PASSWORD', usernameVariable: 'NEXUS_USER')]) {
                                sh "echo ${NEXUS_PASSWORD} | docker login -u ${NEXUS_USER} --password-stdin http://nexus:8083"
                                sh "docker build -t nexus:8083/spring-petclinic:${env.GIT_COMMIT_SHORT} ."
                                sh "docker push nexus:8083/spring-petclinic:${env.GIT_COMMIT_SHORT}"
                            }
                        }
                    }
                }
            }
        }

        stage('Build_b_and_push') {
            when { branch 'main' }
            steps {
                withCredentials([usernamePassword(credentialsId: 'nexus-creds', passwordVariable: 'NEXUS_PASSWORD', usernameVariable: 'NEXUS_USER')]) {
                    script {
                        // Budujemy obraz dla gałęzi main
                        sh "docker build -t nexus:8082/spring-petclinic:latest ."
                        
                        // Logowanie i Push na port 8082
                        sh "echo ${NEXUS_PASSWORD} | docker login -u ${NEXUS_USER} --password-stdin http://nexus:8082"
                        sh "docker push nexus:8082/spring-petclinic:latest"
                    }
                }
            }
        }
    }
}