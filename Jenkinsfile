pipeline {
    agent any

    tools {
        // Tu configuración actual
        nodejs 'node-20'
    }

    stages {
        stage('1. Checkout Code') {
            steps {
                cleanWs()
                // Tu repositorio
                git url: 'https://github.com/Pauetisdev/spring-petclinic-angular.git', branch: 'master'
            }
        }

        stage('2. Install & Test') {
            steps {
                // FIX CRÍTICO: Cambiamos 'npm ci' por 'npm install --force'
                // Esto ignora la incompatibilidad de versiones con Node 20
                sh "npm install --force"
                
                // Ejecuta tests
                sh "npm run test -- --watch=false --code-coverage"
                
                // Guarda el reporte
                archiveArtifacts artifacts: 'coverage/**', allowEmptyArchive: true
            }
        }

        stage('3. SonarQube Scan') {
            environment {
                SONAR_TOKEN = credentials('sonar-token')
            }
            steps {
                withSonarQubeEnv('LocalSonar') {
                    sh "sonar-scanner"
                }
            }
        }

        stage('4. Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }
}