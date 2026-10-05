pipeline {

    agent any

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/sarra111/DevOps-AppGestionDesProjets.git'
            }
        }

        stage('Compile Backend') {
            steps {
                sh '''
                    cd backend
                    chmod +x mvnw
                    ./mvnw compile
                '''
            }
        }

        stage('Tests dynamiques Backend') {
            steps {
                sh '''
                    echo "Starting MySQL..."
                    docker compose up -d mysql

                    echo "Waiting for MySQL..."

                    until docker run --rm \
                        --network devops-appgestiondesprojets_project-network \
                        mysql:8.0 \
                        mysqladmin ping -h mysql -uroot -proot --silent
                    do
                        echo "MySQL is not ready yet..."
                        sleep 3
                    done

                    echo "MySQL is ready!"

                    echo "Running Maven tests..."

                    docker run --rm \
                        --network devops-appgestiondesprojets_project-network \
                        -v "$WORKSPACE/backend:/app" \
                        -w /app \
                        eclipse-temurin:17-jdk \
                        ./mvnw test
                '''
            }
        }

        stage('SonarQube') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                        cd backend
                        ./mvnw sonar:sonar -Dsonar.token=$SONAR_AUTH_TOKEN
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Package Backend') {
            steps {
                sh '''
                    cd backend
                    ./mvnw package -DskipTests
                '''
            }
        }

        stage('Build Docker Backend') {
            steps {
                sh '''
                    docker build \
                        -t project-backend:1.0 \
                        ./backend
                '''
            }
        }

        stage('Deploy Backend') {
            steps {
                sh '''
                    docker compose up -d --force-recreate
                '''
            }
        }

        stage('Compose') {
            steps {
                sh '''
                    docker compose config
                '''

                sh '''
                    docker compose ps
                '''
            }
        }

        stage('Test Backend Application') {
            steps {
                sh '''
                    echo "Waiting for Backend..."
                    sleep 15

                    echo "Testing Backend API..."

                    curl -f http://localhost:8083/equipe/all
                '''
            }
        }
    }
}
