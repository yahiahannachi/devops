pipeline {
    agent any
tools {
    maven 'M2_HOME'
}
    environment {
        SONAR_PROJECT_KEY = 'devops-project'
    }
    stages {
        stage('GIT') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/yahiahannachi/devops.git'
            }
        }
        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh """
                    mvn sonar:sonar \
                    -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                    -Dsonar.projectName=DevOpsProject
                    """
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t yahiahannachi/devops-project:1.0.0 .'
            }
        }
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-hub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push yahiahannachi/devops-project:1.0.0
                    '''
                }
            }
        }
  stage('Kubernetes Deploy') {
            steps {
                sh '''
                echo "=== Création du Namespace devops si nécessaire ==="
                kubectl create namespace devops --dry-run=client -o yaml | kubectl apply -f -

                echo "=== Déploiement des Manifests k8s ==="
                kubectl apply -f k8s/mysql-deployment.yaml
                kubectl apply -f k8s/sonarqube-deployment.yaml
                kubectl apply -f k8s/spring-deployment.yaml

                echo "=== Forcer la mise à jour de l'image de l'application ==="
                kubectl rollout restart deployment/studentmang-app -n devops
                '''
            }
        }

        stage('Kubernetes Status') {
            steps {
                sh '''
                echo "=== Vérification de l'état des Pods dans devops ==="
                kubectl get pods -n devops

                echo "=== Vérification des Services exposés ==="
                kubectl get svc -n devops
                '''
            }
        }
    }
    }
}
