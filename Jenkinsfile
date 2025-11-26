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
                // 1. Limpieza TOTAL
                sh "rm -rf node_modules package-lock.json"
                sh "npm cache clean --force"

                // 2. Instalación base del proyecto
                sh "npm install --force --legacy-peer-deps"
                
                // 3. FIX FINAL: Instalamos la versión EXACTA de karma-coverage compatible con Angular 11 (año 2020)
                // La versión moderna falla con Node 20 + Angular viejo. La 2.0.3 funciona.
                sh "npm install karma-coverage@2.0.3 --save-dev --force"

                // 4. Ejecutar tests
                sh "npm run test -- --watch=false --code-coverage --browsers=ChromeHeadless"
                
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