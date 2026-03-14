pipeline {
    agent any
    tools {
        maven 'maven' // Nombre de tu configuración de Maven en Jenkins
        jdk 'jdk21'   // Nombre de tu configuración de JDK 21 en Jenkins
    }
    environment {
        SONAR_TOKEN = credentials('sonar-token-java') // Credencial nueva para este proyecto
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build & Test') {
            steps {
                // Genera el JAR con dependencias y corre los tests con cobertura
                sh 'mvn clean package'
            }
            post {
                always {
                    // Publica los reportes de tests de JUnit en Jenkins
                  junit testResults: '**/target/surefire-reports/*.xml', allowEmptyResults: true
                }
            }
        }
        stage('SonarQube Analysis') {
    steps {
        // 'SonarQube' es el nombre del servidor en la config global de Jenkins
        withSonarQubeEnv('SonarQube') { 
            sh """
            mvn sonar:sonar \
              -Dsonar.projectKey=wot_bot \
              -Dsonar.host.url=http://sonarqube:9000 \
              -Dsonar.login=${SONAR_TOKEN}
            """
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
        stage('Archive Artifacts') {
            steps {
                // Guarda el JAR generado (el gordo que incluye dependencias)
                archiveArtifacts artifacts: 'target/*-jar-with-dependencies.jar', fingerprint: true
            }
        }
    }
}