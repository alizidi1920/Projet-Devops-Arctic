pipeline {
    agent any

    environment {
        SONAR_PROJECT_KEY = 'AliZidi-Devops-Project'
        DOCKER_IMAGE = 'alizidi-arctic-backend'
        DOCKER_TAG = '1.0'
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Récupération du code depuis GitHub...'
                checkout scm
            }
        }

        stage('Build Maven') {
            steps {
                echo 'Build du projet Spring Boot...'
                dir('backend') {
                    sh 'mvn clean package -DskipTests'
                }
            }
        }

        stage('Tests') {
            steps {
                echo 'Exécution des tests...'
                dir('backend') {
                    sh '''
                        mvn test \
                          -Dspring.datasource.url=jdbc:mysql://localhost:3306/test_db?createDatabaseIfNotExist=true \
                          -Dspring.datasource.username=root \
                          -Dspring.datasource.password=root
                    '''
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo 'Analyse SonarQube...'
                withSonarQubeEnv('SonarQube') {
                    dir('backend') {
                        sh 'mvn sonar:sonar -Dsonar.projectKey=${SONAR_PROJECT_KEY}'
                    }
                }
            }
        }

        stage('Docker Build') {
            steps {
                echo 'Construction de l image Docker...'
                dir('backend') {
                    sh 'docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .'
                }
            }
        }

        stage('Docker Push') {
            steps {
                echo 'Push vers Docker Hub...'
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} $DOCKER_USER/${DOCKER_IMAGE}:${DOCKER_TAG}
                        docker push $DOCKER_USER/${DOCKER_IMAGE}:${DOCKER_TAG}
                    '''
                }
            }
        }

    }

    post {
        success {
            echo 'Pipeline terminé avec succès !'
        }
        failure {
            echo 'Pipeline échoué !'
        }
    }
}
