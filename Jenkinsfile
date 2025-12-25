pipeline {
    agent any

    environment {
        APP_NAME    = "realworld-example-app"
        DOCKER_USER = "rizjosel"
        IMAGE_TAG   = "${BUILD_NUMBER}"
        IMAGE_NAME  = "${DOCKER_USER}/${APP_NAME}:${IMAGE_TAG}"
    }

    stages {

        stage('List Workspace') {
            steps {
                sh 'ls -l'
            }
        }

        stage('Build Application (Gradle)') {
            steps {
                // Clean and build the application without running tests
                //sh './gradlew clean build -x test'
                sh './gradlew clean build --warning-mode all'
                // Archive the JAR artifact(s)
                archiveArtifacts artifacts: 'build/libs/*.jar', fingerprint: true
            }
        }

        stage('Build Docker Image') {
            steps {
                // Build image with Docker Hub–compatible name
                sh 'docker build -t ${IMAGE_NAME} .'

                // Optional: save image as artifact
                //sh 'docker save -o ${APP_NAME}-${IMAGE_TAG}.tar ${IMAGE_NAME}'
                //archiveArtifacts artifacts: "${APP_NAME}-${IMAGE_TAG}.tar", fingerprint: true
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
                        docker push ${IMAGE_NAME}
                        docker logout
                    '''
                }
            }
        }
    }
}
