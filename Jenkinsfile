pipeline {
    agent any

    tools {
        maven 'Maven 3.5.2'
        jdk 'jdk1.8.0_151'
    }

    stages {

        // ÉTAPE 1 : Compilation
        stage('Build') {
            steps {
                sh 'mvn compile'
            }
        }

        // ÉTAPE 2 : Tests en parallèle sur 3 nœuds
        stage('Tests') {
            parallel {

                stage('Tests Unitaires') {
                    steps {
                        sh 'mvn test'
                    }
                    post {
                        always {
                            junit 'target/surefire-reports/**/*.xml'
                        }
                    }
                }

                stage('Couverture de Code') {
                    steps {
                        sh 'mvn cobertura:cobertura'
                    }
                    post {
                        always {
                            cobertura coberturaReportFile: 'target/site/cobertura/coverage.xml'
                        }
                    }
                }

                stage('Documentation') {
                    steps {
                        sh 'mvn site'
                    }
                }
            }
        }

        // ÉTAPE 3 : Packaging
        stage('Package') {
            steps {
                sh 'mvn package -DskipTests'
            }
            post {
                success {
                    archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                }
            }
        }

        // ÉTAPE 4 : Déploiement sur Nexus
        stage('Deploy') {
            steps {
                sh 'mvn deploy -DskipTests'
            }
        }
    }

    // Notification par mail en cas d'erreur
    post {
        failure {
            mail to: 'admin@example.com',
                 subject: "❌ Build échoué : ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: "Le build a échoué. Voir les détails : ${env.BUILD_URL}"
        }
        success {
            echo '✅ Pipeline terminé avec succès !'
        }
    }
}
