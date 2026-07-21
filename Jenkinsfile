pipeline {
    agent any

    tools {
        maven 'MAVEN3'
        jdk 'OracleJDK17'
    }

    environment {
        SONAR_SERVER = 'sonarqube'
        NEXUS_URL = '172.31.82.244:8081'
        RELEASE_REPO = 'vprofile-release'
        SNAPSHOT_REPO = 'vprofile-snapshot'
        NEXUS_CREDENTIAL = 'nexuslogin'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
            post {
                success {
                    archiveArtifacts artifacts: 'target/*.war'
                }
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONAR_SERVER}") {
                    sh '''
                    mvn sonar:sonar \
                      -Dsonar.projectKey=vprofile \
                      -Dsonar.projectName=vprofile
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Deploy to Nexus') {
            steps {
                sh 'mvn -s settings.xml deploy -DskipTests'
            }
        }
    }
}
