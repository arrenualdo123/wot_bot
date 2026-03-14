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
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    // Maven usa automáticamente el plugin de sonar sin configurar .properties
                    sh 'mvn sonar:sonar -Dsonar.projectKey=wotb_bot -Dsonar.projectName="WOTB Bot Java"'
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