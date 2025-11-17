pipeline {
    agent any

    environment {
        APP_SERVICE = 'laravel.test'
    }

    stages {
        stage('Preparar Entorno') {
            steps {
                sh 'cp .env.example .env'
                
                // Limpieza y configuración del .env
                sh 'sed -i "/^DB_/d" .env'
                sh 'echo "" >> .env'
                sh 'echo "DB_CONNECTION=mysql" >> .env'
                sh 'echo "DB_HOST=mysql" >> .env'
                sh 'echo "DB_PORT=3306" >> .env'
                sh 'echo "DB_DATABASE=laravel" >> .env'
                sh 'echo "DB_USERNAME=sail" >> .env'
                sh 'echo "DB_PASSWORD=password" >> .env'
                
                // Debug: Mostrar cómo quedó el .env en los logs
                sh 'cat .env'

                // Instalar dependencias
                sh 'docker run --rm -u "$(id -u):$(id -g)" -v $(pwd):/app composer:lts install --ignore-platform-reqs'
                sh 'chmod -R 777 storage bootstrap/cache'
            }
        }

        stage('Iniciar Sail (Docker)') {
            steps {
                sh './vendor/bin/sail up -d'
                echo 'Esperando a que MySQL inicie (40s)...'
                sh 'sleep 40'
                
                // REINICIO TÁCTICO PARA ARREGLAR DNS
                sh './vendor/bin/sail restart laravel.test'
                
                // Prueba de conexión interna
                echo 'Probando resolución de DNS interna...'
                sh './vendor/bin/sail exec laravel.test getent hosts mysql'
            }
        }

        stage('Configuracion Laravel') {
            steps {
                sh './vendor/bin/sail artisan key:generate'
                sh './vendor/bin/sail artisan migrate:fresh --seed'
            }
        }

        stage('Build Frontend (React)') {
            steps {
                sh './vendor/bin/sail npm install'
                sh './vendor/bin/sail npm run build'
            }
        }

        stage('Tests') {
            steps {
                sh './vendor/bin/sail test'
            }
        }
    }

    post {
        always {
            sh './vendor/bin/sail down'
        }
    }
}