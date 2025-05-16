pipeline {
    agent any

    stages {
        stage('Запуск теста') {
            steps {
                sh 'echo "Привет от Jenkins!"'
                sh 'robot --outputdir reports/ tests/test.robot'
            }
        }
    }
}
