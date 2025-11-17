pipeline {
    agent any

    environment {
        // Usamos el nombre del servicio docker de sail, usualmente laravel.test
        APP_SERVICE = 'laravel.test'
    }

    stages {
        stage('Checkout') {
            steps {
                // Descarga el código de GitHub
                checkout scm
            }
        }

        stage('Preparar Entorno') {
            steps {
                sh 'cp .env.example .env'
                // Usamos una imagen de docker temporal para instalar las dependencias la primera vez
                // Nota: Usamos $(id -u) para que los archivos creados pertenezcan a jenkins y no a root
                sh 'docker run --rm -u "$(id -u):$(id -g)" -v $(pwd):/var/www/html -w /var/www/html laravelsail/php82-composer:latest composer install --ignore-platform-reqs'
            }
        }

        stage('Iniciar Sail (Docker)') {
            steps {
                // Levanta los contenedores en modo Detached (-d)
                sh './vendor/bin/sail up -d'
                // Espera un poco a que MySQL arranque
                sh 'sleep 10'
            }
        }

        stage('Configuracion Laravel') {
            steps {
                sh './vendor/bin/sail artisan key:generate'
                sh './vendor/bin/sail artisan migrate:fresh --seed'
            }
        }

        stage('Build Frontend (React/Shadcn)') {
            steps {
                sh './vendor/bin/sail npm install'
                sh './vendor/bin/sail npm run build'
            }
        }

        stage('Tests') {
            steps {
                // Ejecuta las pruebas unitarias
                sh './vendor/bin/sail test'
            }
        }
    }

    post {
        always {
            // Importante: Apagar los contenedores al terminar para limpiar
            sh './vendor/bin/sail down'
        }
    }
}