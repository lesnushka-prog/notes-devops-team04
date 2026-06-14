pipeline {
    agent any

    triggers {
        pollSCM('* * * * *')
    }

    stages {
        stage('Checkout Code') {
            steps {
                cleanWs()
                checkout scm
            }
        }

        stage('Pre-Build Unit Tests') {
            steps {
                echo 'Запускаем unit tests в Python...'
		sh '''
		    python3 -m venv testvenv
		    python3 -m venv --system-site-packages testvenv || true
		    ./testvenv/bin/python -m pip install --upgrade pip
		    ./testvenv/bin/pip install -r backend/requirements.txt
		    ./testvenv/bin/python backend/test_app.py
		'''
            }
        }

        stage('Docker Deploy') {
            steps {
                echo 'Перезапускаем контейнеры команды project_04...'
                sh 'docker compose -p project_04 down'
                sh 'docker compose -p project_04 up -d --build'
            }
        }

        stage('Post-Build Smoke Tests') {
            steps {
                echo 'Проверяем доступность фронтенда...'
                sh '''
                    sleep 5
                    STATUSCODE=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:8004)
                    echo "Frontend HTTP Status Code: $STATUSCODE"
                    if [ "$STATUSCODE" -ne 200 ]; then
                      echo "Ошибка: фронтенд вернул код $STATUSCODE вместо 200!"
                      exit 1
                    fi
                '''

                echo 'Проверяем health endpoint backend через Nginx...'
                sh '''
                    APICODE=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:8004/api/health)
                    echo "Backend API HTTP Status Code: $APICODE"
                    if [ "$APICODE" -ne 200 ]; then
                      echo "Ошибка: API вернул код $APICODE вместо 200!"
                      exit 1
                    fi
                '''
            }
        }
    }
}
