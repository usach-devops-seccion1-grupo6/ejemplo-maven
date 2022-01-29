import groovy.json.JsonSlurperClassic

def jsonParse(def json) {
    new groovy.json.JsonSlurperClassic().parseText(json)
}

pipeline {
    agent any

    environment {
        NEXUS_CREDS = credentials('user-password-nexus')
        TEST_SLEEP = 80
        TEST_URL = "http://localhost:8081/rest/mscovid/test?msg=testing"
        NEXUS_BASE_URL = "http://nexus:8081/repository/devops-usach-nexus/com/devopsusach2020/DevOpsUsach2020/"
        SONARQUBE_KEY = "feature-nexus"
        NEXUS_JAR_VERSION = "1.0.0"
    }

    stages {
        stage("Paso 1: Compliar"){
            steps {
                script {
                    sh "echo 'Compile Code!'"
                    // Run Maven on a Unix agent.
                    sh "mvn clean compile -e"
                }
            }
        }
        stage("Paso 2: Testear"){
            steps {
                script {
                    echo 'Test Code!'
                    // Run Maven on a Unix agent.
                    sh "mvn clean test -e"
                }
            }
        }
        stage("Paso 3: Build .Jar"){
            steps {
                script {
                    echo 'Build .Jar!'
                    // Run Maven on a Unix agent.
                    sh "mvn clean package -e"
                }
            }
            post {
                //record the test results and archive the jar file.
                success {
                    archiveArtifacts artifacts:'build/*.jar'
                }
            }
        }
        stage("Paso 4: Análisis SonarQube"){
            steps {
                withSonarQubeEnv('sonarqube') {
                    echo 'Calling sonar Service in another docker container!'
                    // Run Maven on a Unix agent to execute Sonar.
                    sh "mvn clean verify sonar:sonar -Dsonar.projectKey=${env.SONARQUBE_KEY}"
                }
            }
        }
        stage("Paso 5: Levantar Springboot APP"){
            steps {
                sh "mvn spring-boot:run &"
            }
        }
        stage("Paso 6: Dormir"){
            steps {
                sh "sleep ${env.TEST_SLEEP}"
            }
        }
        stage("Paso 7: Test Alive Service - Testing Application!"){
            steps {
                sh "curl -X GET ${env.TEST_URL}"
            }
        }
        stage("Paso 8: Subir nueva Version"){
            steps {
                nexusPublisher nexusInstanceId: 'nexus',
                    nexusRepositoryId: 'devops-usach-nexus',
                    packages: [
                        [$class: 'MavenPackage',
                            mavenAssetList: [
                                [classifier: '',
                                extension: '.jar',
                                filePath: "${env.WORKSPACE}/build/DevOpsUsach2020-0.0.1.jar"]
                            ],
                    mavenCoordinate: [
                        artifactId: 'DevOpsUsach2020',
                        groupId: 'com.devopsusach2020',
                        packaging: 'jar',
                        version: "${env.NEXUS_JAR_VERSION}"]
                    ]
                ]
            }
        }
        stage("Paso 9: Bajar Nexus"){
            steps {
                // borro archivo
                sh "rm ${env.WORKSPACE}/build/DevOpsUsach2020-0.0.1.jar"
                // obtengo archivo desde nexus
                sh "curl -X GET -u $NEXUS_CREDS_USR:$NEXUS_CREDS_PSW ${env.NEXUS_BASE_URL}${env.NEXUS_JAR_VERSION}/DevOpsUsach2020-${env.NEXUS_JAR_VERSION}.jar -o ${env.WORKSPACE}/build/DevOpsUsach2020-0.0.1.jar"
            }
        }
        stage("Paso 10: Levantar Springboot APP"){
            steps {
                sh 'mvn spring-boot:run &'
            }
        }
        stage("Paso 11: Dormir"){
            steps {
                sh "sleep ${env.TEST_SLEEP}"
            }
        }
        stage("Paso 12: Test Alive Service"){
            steps {
                sh "curl -X GET ${env.TEST_URL}"
            }
        }
    }

    post {
        always {
            echo 'fase always executed post'
        }

        success {
            echo 'fase success'
        }

        failure {
            echo 'fase failure'
        }
    }
}