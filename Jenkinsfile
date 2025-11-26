pipeline {
    agent any

    tools {
        nodejs 'node-20'
    }

    stages {
        stage('1. Checkout Code') {
            steps {
                cleanWs()
                git url: 'https://github.com/Pauetisdev/spring-petclinic-angular.git', branch: 'master'
            }
        }

        stage('2. Install & Test') {
            steps {
                // 1. Limpieza
                sh "rm -rf node_modules package-lock.json"
                sh "npm cache clean --force"

                // 2. Instalación general
                sh "npm install --force --legacy-peer-deps"
                
                // 3. FIX DEL ERROR: Instalamos el reportero de cobertura manualmente
                sh "npm install karma-coverage --save-dev --force"

                // 4. Tests
                sh "npm run test -- --watch=false --code-coverage"
                
                // 5. Guardar reporte
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