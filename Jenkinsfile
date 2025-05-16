pipeline {
    agent any

    stages {
        stage('Установка Robot Framework') {
            steps {
                sh '''
                    # Устанавливаем Python и pip
                    sudo apt-get update
                    sudo apt-get install -y python3 python3-pip
                    
                    # Устанавливаем Robot Framework
                    pip3 install robotframework
                '''
            }
        }

        stage('Запуск теста') {
            steps {
                sh 'echo "Привет от Jenkins!"'
                sh 'robot --outputdir reports/ tests/test.robot'
            }
        }
    }
}
