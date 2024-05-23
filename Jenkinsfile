pipeline {
    // Chọn môi trường để chạy pipeline
    // Bất kỳ mỗi trường nào như ubuntu, windows, docker, ở đây chạy mặc định là docker
    agent any   
    // Tools maven được tạo ở jenkins
    tools { 
        maven 'my-maven' 
    }
    // Biến môi trường để đăng nhập vào mysql
    // Tạo credentials trong jenkins
    environment {
        MYSQL_ROOT_LOGIN = credentials('mysql-root-login')
    }
    stages {

        stage('Build with Maven') {
            steps {
                sh 'mvn --version'
                sh 'java -version'
                sh 'mvn clean package -Dmaven.test.failure.ignore=true'
            }
        }

        stage('Packaging/Pushing image') {

            steps {
                withDockerRegistry(credentialsId: 'dockerhub', url: 'https://index.docker.io/v1/') {
                    sh 'docker build -t tuan2110/springboot .'
                    sh 'docker push tuan2110/springboot'
                }
            }
        }

        stage('Deploy MySQL to DEV') {
            steps {
                echo 'Deploying and cleaning'
                sh 'docker image pull mysql:8.0'
                sh 'docker network create dev || echo "this network exists"'
                sh 'docker container stop tuan2110-mysql || echo "this container does not exist" '
                sh 'echo y | docker container prune '
                sh 'docker volume rm tuan2110-mysql-data || echo "no volume"'

                sh "docker run --name tuan2110-mysql --rm --network dev -v tuan2110-mysql-data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_LOGIN_PSW} -e MYSQL_DATABASE=db_example  -d mysql:8.0 "
                sh 'sleep 20'
                sh "docker exec -i tuan2110-mysql mysql --user=root --password=${MYSQL_ROOT_LOGIN_PSW} < script"
            }
        }

        stage('Deploy Spring Boot to DEV') {
            steps {
                echo 'Deploying and cleaning'
                sh 'docker image pull tuan2110/springboot'
                sh 'docker container stop tuan2110-springboot || echo "this container does not exist" '
                sh 'docker network create dev || echo "this network exists"'
                sh 'echo y | docker container prune '

                sh 'docker container run -d --rm --name tuan2110-springboot -p 8081:8080 --network dev tuan2110/springboot'
            }
        }
    }
    post {
        // Clean after build
        always {
            cleanWs()
        }
    }
}
