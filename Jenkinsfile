pipeline {
    agent any
    environment {
        // SonarQube
        SONAR_QUBE_CREDENTIALS_ID = 'TOKEN1'
        SONAR_QUBE_NAME = 'sonarqube-server'
        
        // Nexus
        NEXUS_REPOSITORY_ID = 'Nexus_customer_app2'
        NEXUS_URL = 'http://3.110.33.225:8081/repository/Nexus_customer_app2/'
        // Tomcat
        TOMCAT_URL = 'http://34.202.205.194:8080/manager/text'
        TOMCAT_CREDENTIALS_ID = 'tomcat-credentials'
        TOMCAT_APP_CONTEXT = 'SimpleCustomerApp'
    }
    tools {
        maven 'MVN_HOME'
        jdk 'jdk21'
    }
    stages {
        stage('Git Clone') {
            steps {
                git branch: 'feature-1.1', url: 'https://github.com/venkatesh221024/venkatesh_simplecutomerapp.git'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv(credentialsId: "${SONAR_QUBE_CREDENTIALS_ID}", installationName: "${SONAR_QUBE_NAME}") {
                    sh 'mvn clean verify sonar:sonar -DskipTests'
                }
            }
        }
        stage('Maven Package') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }
        stage('Deploy to Nexus') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'Nexus-credentials',
                    usernameVariable: 'NEXUS_USER',
                    passwordVariable: 'NEXUS_PASS'
                )]) {
                    writeFile file: 'settings-temp.xml', text: """
                        <settings xmlns="http://maven.apache.org/SETTINGS/1.0.0">
                          <servers>
                            <server>
                              <id>${env.NEXUS_REPOSITORY_ID}</id>
                              <username>${env.NEXUS_USER}</username>
                              <password>${env.NEXUS_PASS}</password>
                            </server>
                          </servers>
                        </settings>
                    """
                    sh 'mvn deploy -DskipTests --settings settings-temp.xml'
                }
            }
        }
        stage('Deploy to Tomcat') {
            steps {
                script {
                    def warFile = findFiles(glob: 'target/*.war')[0]
                    if (warFile) {
                        sh "mv ${warFile.path} target/${env.TOMCAT_APP_CONTEXT}.war"
                        step([
                            $class: 'DeployPublisher',
                            adapters: [[
                                $class: 'Tomcat9xAdapter',
                                credentialsId: "${TOMCAT_CREDENTIALS_ID}",
                                url: "${TOMCAT_URL}"
                            ]],
                            war: "target/${env.TOMCAT_APP_CONTEXT}.war",
                            contextPath: "${env.TOMCAT_APP_CONTEXT}"
                        ])
                    } else {
                        error 'WAR file not found!'
                    }
                }
            }
        }
    }
    post {
        always {
            echo 'Pipeline execution completed.'
        }
        success {
            echo ':white_tick: Pipeline succeeded.'
        }
        failure {
            echo ':x: Build or Deployment Failed!'
        }
    }
}
