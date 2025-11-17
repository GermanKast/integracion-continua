pipeline {
    agent any

    environment {
        // Definimos el nombre del servicio principal
        APP_SERVICE = 'laravel.test'
    }

    stages {
        stage('Preparar Entorno') {
            steps {
                // 1. Copiar el archivo de entorno
                sh 'cp .env.example .env'
                
                // 2. Instalar dependencias de PHP (Backend)
                // Usamos la imagen oficial de Composer.
                // Mapeamos el usuario actual (id -u) para evitar problemas de permisos root.
                sh 'docker run --rm -u "$(id -u):$(id -g)" -v $(pwd):/app composer:lts install --ignore-platform-reqs'
                
                // 3. Arreglar permisos (CRÍTICO EN LINUX)
                // Damos permisos de escritura a las carpetas de logs y cache
                sh 'chmod -R 777 storage bootstrap/cache'
            }
        }

        stage('Iniciar Sail (Docker)') {
            steps {
                // 4. Iniciar los contenedores
                sh './vendor/bin/sail up -d'
                
                // 5. Esperar a que MySQL despierte (Damos 30 segundos de seguridad)
                echo 'Esperando a que la Base de Datos inicie...'
                sh 'sleep 30'
            }
        }

        stage('Configuracion Laravel') {
            steps {
                // 6. Generar la llave de encriptación (Esto rellena el APP_KEY en .env)
                sh './vendor/bin/sail artisan key:generate'
                
                // 7. Migrar la base de datos (Crear tablas)
                sh './vendor/bin/sail artisan migrate:fresh --seed'
            }
        }

        stage('Build Frontend (React)') {
            steps {
                // 8. Instalar dependencias de JS y compilar
                sh './vendor/bin/sail npm install'
                sh './vendor/bin/sail npm run build'
            }
        }

        stage('Tests') {
            steps {
                // 9. Ejecutar pruebas
                sh './vendor/bin/sail test'
            }
        }
    }

    post {
        always {
            // 10. Apagar todo al finalizar (sea éxito o error)
            sh './vendor/bin/sail down'
        }
    }
}