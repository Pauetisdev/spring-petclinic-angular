pipeline {
    agent any

    tools {
        // Usamos node-20 como tienes configurado
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
                // 1. Limpieza radical: borramos carpetas y caché para empezar de cero
                sh "rm -rf node_modules package-lock.json"
                sh "npm cache clean --force"

                // 2. Instalación forzosa ignorando dependencias antiguas
                sh "npm install --force --legacy-peer-deps"
                
                // 3. Instalación manual de los plugins necesarios para evitar el error "Can not load reporter"
                // Instalamos versiones compatibles
                sh "npm install karma-coverage@2.0.3 --save-dev --force"
                sh "npm install puppeteer --save-dev --force"

                // 4. Parcheamos el archivo karma.conf.js para registrar el plugin
                sh "find . -name karma.conf.js -exec sed -i \"s|plugins: \\[|plugins: [ require('karma-coverage'),|\" {} +"

                // 5. Configuramos la variable de Chrome y lanzamos los tests
                script {
                    // Le decimos a Karma que use el Chrome que acabamos de bajar con Puppeteer
                    sh """
                        export CHROME_BIN=\$(node -p "require('puppeteer').executablePath()")
                        npm run test -- --watch=false --code-coverage --browsers=ChromeHeadless
                    """
                }
                
                // 6. Guardamos el reporte HTML resultante
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