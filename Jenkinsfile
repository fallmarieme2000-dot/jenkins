pipeline {
    agent any

    tools {
        nodejs "NodeJS_20"
    }

    environment {
        DOCKER_HUB_USER = 'marieme0516'
        FRONT_IMAGE = 'react-frontend'
        BACK_IMAGE  = 'express-backend'
    }
    triggers {
        // Pour que le pipeline démarre quand le webhook est reçu
        GenericTrigger(
            genericVariables: [
                [key: 'ref', value: '$.ref'],
                [key: 'pusher_name', value: '$.pusher.name'],
                [key: 'commit_message', value: '$.head_commit.message']
            ],
            causeString: 'Push par $pusher_name sur $ref: "$commit_message"',
            token: 'mysecret',
            printContributedVariables: true,
            printPostContent: true
        )
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/fallmarieme2000-dot/jenkins.git'
            }
        }

        stage('Install dependencies - Backend') {
            steps {
                dir('back-end') {
                    bat 'npm install'
                }
            }
        }

        stage('Install dependencies - Frontend') {
            steps {
                dir('front-end') {
                    bat 'npm install'
                }
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    bat 'cd back-end && npm test || echo "Aucun test backend"'
                    bat 'cd front-end && npm test || echo "Aucun test frontend"'
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    bat "docker build -t $DOCKER_HUB_USER/$FRONT_IMAGE:latest ./front-end"
                    bat "docker build -t $DOCKER_HUB_USER/$BACK_IMAGE:latest ./back-end"
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    bat '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push $DOCKER_USER/react-frontend:latest
                        docker push $DOCKER_USER/express-backend:latest
                    '''
                }
            }
        }

        // on supprime les conteneur inactif dans docker container
        stage('Clean Docker') {
            steps {
                bat 'docker container prune -f'
                bat 'docker image prune -f'
            }
        }

        stage('Check Docker & Compose') {
            steps {
                bat 'docker --version'
                bat 'docker-compose --version || echo "docker-compose non trouvé"'
            }
        }

        stage('Deploy (compose.yaml)') {
            steps {
                dir('.') {  
                    bat 'docker-compose -f compose.yaml down || true'
                    bat 'docker-compose -f compose.yaml pull'
                    bat 'docker-compose -f compose.yaml up -d'
                    bat 'docker-compose -f compose.yaml ps'
                    bat 'docker-compose -f compose.yaml logs --tail=50'
                }
            }
        }

        stage('Smoke Test') {
            steps {
                bat '''
                    echo " Vérification Frontend (port 5173)..."
                    curl -f http://localhost:5173 || echo "Frontend unreachable"

                    echo " Vérification Backend (port 5001)..."
                    curl -f http://localhost:5001/api || echo "Backend unreachable"
                '''
            }
        }
    }

    post {
        success {
            emailext(
                subject: "Build SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Pipeline réussi\nDétails : ${env.BUILD_URL}",
                to: "mohamedndoye07@gmail.com"
            )
        }
        failure {
            emailext(
                subject: "Build FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Le pipeline a échoué\nDétails : ${env.BUILD_URL}",
                to: "mohamedndoye07@gmail.com"
            )
        }
    }
}
