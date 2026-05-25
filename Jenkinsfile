pipeline {
    agent any

    options {
        timestamps()
    }

    stages {
        stage('环境检查') {
            steps {
                bat '''
                java -version
                mvn -version
                git --version
                '''
            }
        }

        stage('Maven构建') {
            steps {
                bat 'mvn -B -DskipTests clean install'
            }
        }

        stage('PMD代码检查') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
                    bat 'mvn -B -DskipTests org.apache.maven.plugins:maven-pmd-plugin:3.21.2:pmd'
                }
            }
        }

        stage('运行测试') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
                    bat 'mvn -B -Dmaven.test.failure.ignore=true test'
                }
            }
        }

        stage('生成测试报告') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
                    bat 'mvn -B surefire-report:report-only'
                }
            }
        }

        stage('生成JavaDoc文档') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
                    bat 'mvn -B -DskipTests -Dmaven.javadoc.failOnError=false org.apache.maven.plugins:maven-javadoc-plugin:3.6.3:javadoc org.apache.maven.plugins:maven-javadoc-plugin:3.6.3:jar'
                }
            }
        }
    }

    post {
        always {
            junit allowEmptyResults: true, testResults: '**/target/surefire-reports/*.xml'

            archiveArtifacts artifacts: '''
                **/target/*.jar,
                **/target/*.war,
                **/target/site/**/*.html,
                **/target/apidocs/**,
                **/target/*-javadoc.jar,
                **/target/surefire-reports/**
            ''', allowEmptyArchive: true
        }

        success {
            echo 'Teedy Jenkins流水线执行完成'
        }

        unstable {
            echo 'Teedy Jenkins流水线执行完成，但部分检查或测试存在问题'
        }

        failure {
            echo 'Teedy Jenkins流水线执行失败，请查看控制台日志'
        }
    }
}