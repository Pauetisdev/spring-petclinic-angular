pipeline {
    agent any

    tools {
        // Mantenemos tu configuración de node-20
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
                // FIX 1: Borramos carpeta node_modules por si tiene permisos corruptos
                sh "rm -rf node_modules"

                // FIX 2: Usamos install --force para evitar el error de versión y permisos de 'ci'
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