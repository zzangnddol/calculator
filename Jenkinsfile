pipeline {
    agent any
    triggers {
        pollSCM('* * * * *')
    }
    stages {
        stage("Compile") {
            steps {
                sh "./gradlew compileJava"
            }
        }
        stage("Unit test") {
            steps {
                sh "./gradlew test"
            }
        }
        stage("Code coverage") {
            steps {
                sh "./gradlew jacocoTestReport"
                publishHTML (target: [
                    reportDir: 'build/reports/jacoco/test/html',
                    reportFiles: 'index.html',
                    reportName: "JaCoCo Report"
                ])
                sh "./gradlew jacocoTestCoverageVerification"
            }
        }
        stage("Static code analysis") {
            steps {
                sh "./gradlew checkstyleMain"
            }
            post {
                always {
                    publishHTML (target: [
                        reportDir: 'build/reports/checkstyle/',
                        reportFiles: 'main.html',
                        reportName: "Checkstyle Report"
                    ])
                }
            }
        }
        stage("Package") {
            steps {
                sh "./gradlew build"
            }
        }
        stage("Docker build") {
            steps {
                // sleep 600
                sh "docker build -t ybsong/calculator ."
            }
        }
        stage("Restart") {
          steps {
            sh 'docker ps -q --filter name=calculator | grep -q . && docker stop calculator && docker rm calculator'
            sh 'docker run -d --name calculator -p 8080:8080 ybsong/calculator'
          }
        }
        stage("Finish") {
          steps {
            sh 'docker images -qf dangling=true | xargs -I{} docker rmi {}'
          }
        }
    }
}