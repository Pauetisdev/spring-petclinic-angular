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
                // 1. Limpieza nuclear
                sh "rm -rf node_modules package-lock.json"
                sh "npm cache clean --force"

                // 2. Instalación base forzada
                sh "npm install --force --legacy-peer-deps"
                
                // 3. FIX MAESTRO: Instalamos manualmente los plugins de cobertura y actualizamos jasmine-core
                // Esto arregla la incompatibilidad del reporter "coverage"
                sh "npm install karma-coverage karma-coverage-istanbul-reporter jasmine-core --save-dev --force"

                // 4. Ejecutar tests (Usamos ChromeHeadless para evitar errores gráficos en Docker)
                // Si ChromeHeadless falla, Jenkins probará Chrome normal, pero este es más seguro.
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