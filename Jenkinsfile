pipeline {
    agent any

    tools {
        jdk 'jdk-17'
    }

    stages {
        stage('pipeline_mr') {
            when { not { branch 'main' } }
            stages {
                stage('Checkstyle_mr') {
                    steps {
                        sh './gradlew checkstyleMain'
                    }
                    post {
                        always { 
                            archiveArtifacts artifacts: 'build/reports/checkstyle/*.html', allowEmptyArchive: true
                        }
                    }
                }
                stage('Test_mr') {
                    steps {
                        sh './gradlew test'
                    }
                }
                stage('Build_mr_and_push') {
                    steps {
                        sh './gradlew bootJar -x test'
                        script {
                            env.GIT_COMMIT_SHORT = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                            
                            withCredentials([usernamePassword(credentialsId: 'nexus-creds', passwordVariable: 'NEXUS_PASSWORD', usernameVariable: 'NEXUS_USER')]) {
                                // 127.0.0.1 wymusza na Dockerze ominięcie sprawdzania certyfikatów HTTPS
                                sh 'echo $NEXUS_PASSWORD | docker login -u $NEXUS_USER --password-stdin 127.0.0.1:8083'
                                sh "docker build -t 127.0.0.1:8083/spring-petclinic:${env.GIT_COMMIT_SHORT} ."
                                sh "docker push 127.0.0.1:8083/spring-petclinic:${env.GIT_COMMIT_SHORT}"
                            }
                        }
                    }
                }
            }
        }

        stage('Build_main_and_push') {
            when { branch 'main' }
            steps {
                sh './gradlew bootJar -x test'
                
                withCredentials([usernamePassword(credentialsId: 'nexus-creds', passwordVariable: 'NEXUS_PASSWORD', usernameVariable: 'NEXUS_USER')]) {
                    script {
                        sh "docker build -t 127.0.0.1:8082/spring-petclinic:latest ."
                        sh 'echo $NEXUS_PASSWORD | docker login -u $NEXUS_USER --password-stdin 127.0.0.1:8082'
                        sh "docker push 127.0.0.1:8082/spring-petclinic:latest"
                    }
                }
            }
        }
    }
}