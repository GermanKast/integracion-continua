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
                // Copia el archivo de entorno y genera la key
                sh 'cp .env.example .env'
                // Instala dependencias de PHP usando una imagen temporal de composer
                // Esto es necesario para tener el archivo 'sail' disponible
                //sh 'docker run --rm -v $(pwd):/var/www/html -w /var/www/html laravel/sail:latest-php8.2 composer install --ignore-platform-reqs'
                // * CORRECCIÓN * Alternativa usando la imagen oficial de composer
                sh 'docker run --rm -v $(pwd):/app composer install --ignore-platform-reqs'
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