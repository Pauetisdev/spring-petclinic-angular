pipeline {
    agent any

    tools {
        // Configuración exacta que pediste: node-20
        nodejs 'node-20'
    }

    stages {
        stage('1. Checkout Code') {
            steps {
                cleanWs()
                // Descarga TU repositorio del frontend
                git url: 'https://github.com/Pauetisdev/spring-petclinic-angular.git', branch: 'master'
            }
        }

        stage('2. Install & Test') {
            steps {
                // 1. Instalar dependencias
                sh "npm ci"
                
                // 2. Ejecutar tests y generar cobertura (crea la carpeta /coverage)
                sh "npm run test -- --watch=false --code-coverage"
                
                // 3. GUARDAR EL REPORTE HTML (Importante: /** guarda archivos y estilos)
                archiveArtifacts artifacts: 'coverage/**', allowEmptyArchive: true
            }
        }

        stage('3. SonarQube Scan') {
            environment {
                SONAR_TOKEN = credentials('sonar-token')
            }
            steps {
                withSonarQubeEnv('LocalSonar') {
                    // Ejecuta el escáner. 
                    // Requiere que tengas el archivo sonar-project.properties en la raíz del repo
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