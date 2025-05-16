pipeline {
    agent {
        docker {
            image 'python:3.9-alpine'
            args '-v $WORKSPACE:/app'
        }
    }

    stages {
        stage('Установка Robot Framework') {
            steps {
                sh '''
                    # Установка зависимостей
                    apk update
                    apk add --no-cache gcc musl-dev linux-headers
                    pip install robotframework
                '''
            }
        }

        stage('Запуск теста') {
            steps {
                sh 'robot --outputdir reports/ tests/test.robot'
            }
        }
    }
}
