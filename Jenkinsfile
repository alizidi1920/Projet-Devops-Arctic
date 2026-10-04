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

        stage('Start MySQL') {
            steps {
                echo 'Démarrage de MySQL pour les tests...'
                sh '''
                    docker rm -f mysql-test || true
                    docker run -d \
                        --name mysql-test \
                        -e MYSQL_ROOT_PASSWORD=root \
                        -e MYSQL_DATABASE=arctic \
                        -e MYSQL_USER=user \
                        -e MYSQL_PASSWORD=password \
                        -p 3307:3306 \
                        mysql:8
                    echo "Attente démarrage MySQL..."
                    sleep 20
                '''
            }
        }

        stage('Tests') {
            steps {
                echo 'Exécution des tests...'
                dir('backend') {
                    sh '''
                        mvn test \
                          -Dspring.datasource.url=jdbc:mysql://localhost:3307/arctic \
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
        always {
            echo 'Nettoyage MySQL de test...'
            sh 'docker rm -f mysql-test || true'
        }
        success {
            echo 'Pipeline terminé avec succès !'
        }
        failure {
            echo 'Pipeline échoué !'
        }
    }
}
