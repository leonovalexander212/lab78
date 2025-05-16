pipeline {
    agent {
        docker {
            image 'jenkins/jenkins:lts'
            args '-v /var/run/docker.sock:/var/run/docker.sock -v jenkins_home:/var/jenkins_home'
        }
    }

    stages {
        stage('Запуск QEMU с OpenBMC') {
            steps {
                sh '''
                    qemu-system-arm \
                        -M romulus-bmc \
                        -drive file=./openbmc-image,format=raw \
                        -nographic \
                        -monitor telnet::5555,server,nowait \
                        -serial mon:stdio &
                    sleep 60
                '''
            }
        }

        stage('Запуск автотестов') {
            steps {
                sh 'robot --outputdir reports/autotests tests/autotests.robot'
            }
            post {
                always {
                    archiveArtifacts artifacts: 'reports/autotests/**/*'
                }
            }
        }

        stage('WebUI тесты') {
            steps {
                sh 'robot --outputdir reports/webtests tests/webtests.robot'
            }
            post {
                always {
                    archiveArtifacts artifacts: 'reports/webtests/**/*'
                }
            }
        }

        stage('Нагрузочное тестирование') {
            steps {
                sh 'locust -f tests/loadtest.py --headless -u 100 -r 10 --run-time 1m'
                archiveArtifacts artifacts: 'loadtest_stats.csv'
            }
        }

        stage('Профилирование QEMU') {
            steps {
                sh 'perf stat -e cycles,instructions,cache-misses -o perf_report.txt qemu-system-arm ...'
                sh 'vmstat 1 60 > vmstat.log'
                sh 'nmon -s1 -c 60 -f -T'
            }
            post {
                always {
                    archiveArtifacts artifacts: 'perf_report.txt, vmstat.log, *.nmon'
                }
            }
        }
    }
}
