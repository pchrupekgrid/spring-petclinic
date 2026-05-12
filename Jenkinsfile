pipeline {
    agent any

    environment {
        // Wymuszamy bezpośrednie połączenie z Twoim Makiem
        DOCKER_HOST = 'unix:///var/run/docker.sock'
        // Wyłączamy sprawdzanie certyfikatów (to naprawi błąd ca.pem)
        DOCKER_TLS_VERIFY = '0'
        // Czyścimy ścieżkę certyfikatów, żeby nie szukał plików w .docker
        DOCKER_CERT_PATH = ''
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
                        // Sprawdzamy połączenie - teraz 'Server' powinien się pokazać poprawnie
                        sh "docker version"
                        
                        // Budowa obrazu (korzysta z silnika Twojego Maca M1)
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