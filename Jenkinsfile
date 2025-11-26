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

                // 2. Instalación forzada de dependencias
                sh "npm install --force --legacy-peer-deps"
                
                // 3. Instalamos el plugin de cobertura
                sh "npm install karma-coverage --save-dev --force"

                // 4. TRUCO FINAL (PATCH): Modificamos karma.conf.js para registrar el plugin a la fuerza
                // Buscamos dónde empieza la lista 'plugins: [' y le metemos 'require("karma-coverage"),'
                sh "find . -name karma.conf.js -exec sed -i \"s|plugins: \\[|plugins: [ require('karma-coverage'),|\" {} +"

                // 5. Ejecutar tests (ahora sí encontrará el reporter)
                sh "npm run test -- --watch=false --code-coverage --browsers=ChromeHeadless"
                
                // 6. Guardar reporte
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