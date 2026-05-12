pipeline {
    agent any

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
                            // Pobranie krótkiego hasha commita
                            env.GIT_COMMIT_SHORT = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                            
                            // Logowanie do Nexusa (Port 8083 dla MR) i wysyłka
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
                    // Logowanie do Nexusa (Port 8082 dla Main) i wysyłka obrazu z tagiem latest
                    docker.withRegistry('http://nexus:8082', 'nexus-creds') {
                        def image = docker.build("spring-petclinic:latest")
                        image.push()
                    }
                }
            }
        }
    }
}