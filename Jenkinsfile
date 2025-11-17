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
                
                // --- ESTRATEGIA LIMPIA PARA .ENV ---
                // En lugar de buscar y reemplazar, borramos toda config de DB vieja
                // y agregamos la nueva al final. Es a prueba de errores.
                
                // 1. Borrar cualquier línea que empiece por DB_
                sh 'sed -i "/^DB_/d" .env'
                
                // 2. Inyectar la configuración limpia para Docker/Sail
                sh 'echo "" >> .env'
                sh 'echo "DB_CONNECTION=mysql" >> .env'
                sh 'echo "DB_HOST=mysql" >> .env'
                sh 'echo "DB_PORT=3306" >> .env'
                sh 'echo "DB_DATABASE=laravel" >> .env'
                sh 'echo "DB_USERNAME=sail" >> .env'
                sh 'echo "DB_PASSWORD=password" >> .env'
                
                // (Opcional) Verificar visualmente en los logs cómo quedó el archivo
                sh 'grep "DB_" .env' 
                // -----------------------------------

                // 3. Instalar dependencias de PHP
                sh 'docker run --rm -u "$(id -u):$(id -g)" -v $(pwd):/app composer:lts install --ignore-platform-reqs'
                
                // 4. Arreglar permisos
                sh 'chmod -R 777 storage bootstrap/cache'
            }
        }

        stage('Iniciar Sail (Docker)') {
            steps {
                sh './vendor/bin/sail up -d'
                echo 'Esperando a que la Base de Datos inicie (40s)...'
                sh 'sleep 40'
            }
        }

        stage('Configuracion Laravel') {
            steps {
                sh './vendor/bin/sail artisan key:generate'
                
                // Intentamos la migración. Si falla, Sail reintentará la conexión.
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