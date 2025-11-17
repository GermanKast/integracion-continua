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
                
                // 2. Configuración limpia del .env
                sh 'sed -i "/^DB_/d" .env'
                sh 'echo "" >> .env'
                sh 'echo "DB_CONNECTION=mysql" >> .env'
                sh 'echo "DB_HOST=mysql" >> .env'
                sh 'echo "DB_PORT=3306" >> .env'
                sh 'echo "DB_DATABASE=laravel" >> .env'
                sh 'echo "DB_USERNAME=sail" >> .env'
                sh 'echo "DB_PASSWORD=password" >> .env'

                // 3. Instalar dependencias de PHP
                sh 'docker run --rm -u "$(id -u):$(id -g)" -v $(pwd):/app composer:lts install --ignore-platform-reqs'
                
                // 4. Arreglar permisos
                sh 'chmod -R 777 storage bootstrap/cache'
            }
        }

        stage('Iniciar Sail (Limpieza Total)') {
            steps {
                // --- PASO CRÍTICO: DESTRUCCIÓN TOTAL ---
                // Borramos contenedores y VOLÚMENES (-v) viejos para evitar redes zombis
                // El '|| true' evita que falle si no había nada corriendo
                sh './vendor/bin/sail down -v || true'
                
                // Iniciamos forzando recreación de contenedores
                sh './vendor/bin/sail up -d --force-recreate'
                
                // Esperamos 60s porque al borrar volúmenes, MySQL tarda más en arrancar la primera vez
                echo 'Esperando a que MySQL inicialice desde cero (60s)...'
                sh 'sleep 60'
            }
        }

        stage('Configuracion Laravel') {
            steps {
                sh './vendor/bin/sail artisan key:generate'
                
                // Ejecutamos la migración
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
            // Limpieza final
            sh './vendor/bin/sail down -v'
        }
    }
}