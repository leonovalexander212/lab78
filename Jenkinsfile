pipeline {
    agent {
        docker {
            // Используем образ с Ubuntu и предустановленными зависимостями
            image 'ubuntu:22.04'
            args '--privileged -v /var/run/docker.sock:/var/run/docker.sock -v $WORKSPACE:/workspace'
            registryUrl 'https://registry.hub.docker.com'
            reuseNode true
        }
    }

    environment {
        DEBIAN_FRONTEND = 'noninteractive'
    }

    stages {
        stage('Установка зависимостей') {
            steps {
                sh '''
                    apt-get update && apt-get install -y \
                    qemu-system-arm \
                    robotframework \
                    python3-locust \
                    linux-tools-common \
                    nmon \
                    && rm -rf /var/lib/apt/lists/*
                '''
            }
        }

        stage('Запуск QEMU') {
            steps {
                sh '''
                    qemu-system-arm \
                        -M romulus-bmc \
                        -drive file=/workspace/openbmc-image,format=raw \
                        -nographic \
                        -monitor telnet::5555,server,nowait \
                        -serial mon:stdio &
                    sleep 30  # Ожидание инициализации
                '''
            }
        }

        stage('Автотесты') {
            steps {
                sh 'robot --outputdir /workspace/reports/autotests /workspace/tests/autotests.robot'
            }
            post {
                always {
                    archiveArtifacts artifacts: 'reports/autotests/**/*'
                }
            }
        }

        stage('WebUI тесты') {
            steps {
                sh 'robot --outputdir /workspace/reports/webtests /workspace/tests/webtests.robot'
            }
            post {
                always {
                    archiveArtifacts artifacts: 'reports/webtests/**/*'
                }
            }
        }

        stage('Нагрузочное тестирование') {
            steps {
                sh 'locust -f /workspace/tests/loadtest.py --headless -u 100 -r 10 --run-time 1m --csv=/workspace/loadtest'
                archiveArtifacts artifacts: 'loadtest_stats.csv'
            }
        }

        stage('Профилирование') {
            steps {
                sh 'perf stat -e cycles,instructions,cache-misses -o /workspace/perf_report.txt qemu-system-arm ...'
                sh 'vmstat 1 60 > /workspace/vmstat.log'
                sh 'nmon -s1 -c 60 -f -T -F /workspace/nmon_report.nmon'
            }
            post {
                always {
                    archiveArtifacts artifacts: 'perf_report.txt, vmstat.log, *.nmon'
                }
            }
        }
    }

    post {
        always {
            sh 'pkill qemu-system-arm'  // Остановка QEMU
        }
    }
}
