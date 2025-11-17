pipeline {
    agent any

    environment {
        APP_SERVICE = 'laravel.test'
    }

    stages {
        stage('Preparar Entorno') {
            steps {
                // 1. Copiar el archivo de entorno
                sh 'cp .env.example .env'
                
                // --- NUEVO BLOQUE: FORZAR CONFIGURACIÓN MYSQL ---
                // Laravel 12 trae SQLite por defecto. Aquí lo cambiamos a MySQL para Sail.
                
                // Cambiar conexión de sqlite a mysql
                sh 'sed -i "s/DB_CONNECTION=sqlite/DB_CONNECTION=mysql/g" .env'
                
                // Cambiar el host: De 127.0.0.1 a "mysql" (nombre del contenedor de docker)
                // El comando intenta descomentar la linea (#) si existe, o cambiarla si no tiene #
                sh 'sed -i "s/# DB_HOST=127.0.0.1/DB_HOST=mysql/g" .env'
                sh 'sed -i "s/DB_HOST=127.0.0.1/DB_HOST=mysql/g" .env'
                
                // Configurar puerto
                sh 'sed -i "s/# DB_PORT=3306/DB_PORT=3306/g" .env'
                
                // Configurar base de datos, usuario y contraseña (Valores por defecto de Sail)
                sh 'sed -i "s/# DB_DATABASE=laravel/DB_DATABASE=laravel/g" .env'
                sh 'sed -i "s/# DB_USERNAME=root/DB_USERNAME=sail/g" .env'
                sh 'sed -i "s/# DB_PASSWORD=/DB_PASSWORD=password/g" .env'
                // -----------------------------------------------

                // 2. Instalar dependencias de PHP
                sh 'docker run --rm -u "$(id -u):$(id -g)" -v $(pwd):/app composer:lts install --ignore-platform-reqs'
                
                // 3. Arreglar permisos
                sh 'chmod -R 777 storage bootstrap/cache'
            }
        }

        stage('Iniciar Sail (Docker)') {
            steps {
                sh './vendor/bin/sail up -d'
                echo 'Esperando a que la Base de Datos inicie...'
                sh 'sleep 30'
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