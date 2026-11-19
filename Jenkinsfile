pipeline {
    agent any
    tools {
        maven 'MAVEN_3_10'
        jdk 'JDK_26'
    }
    environment {
        REGISTRY_USER   = "estefanosolis" // Cambia por tu usuario real de Docker Hub
        IMAGE_NAME      = "retail-store-u202314354"
        TAG             = "${env.BUILD_NUMBER}" // Usa el número de ejecución de Jenkins como versión
    }

    stages {
        stage ('Compile Project') {
            steps {
                withMaven(maven : 'MAVEN_3_10') {
                    sh 'mvn clean compile'
                }
            }
        }


stage('Checkstyle') {
    steps {
        withMaven(maven: 'MAVEN_3_10') {
            sh 'mvn checkstyle:check'
        }
    }
}

stage('Validate Unit Tests') {
    steps {
        withMaven(maven: 'MAVEN_3_10') {
            sh 'mvn test'
        }
    }
}

        stage('Validate Test Coverage') {
            steps {
                withMaven(maven: 'MAVEN_3_10') {
                    withSonarQubeEnv('SonarQubeServer') {
                        sh 'mvn clean verify sonar:sonar'
                    }
                    timeout(time: 5, unit: 'MINUTES') {
                        waitForQualityGate abortPipeline: true
                    }
                }
            }
        }

        stage('Docker') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'DOCKER_HUB_CREDENTIALS', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    script {
                        echo "Iniciando sesión en Docker Hub..."
                        sh "echo '${DOCKER_PASS}' | docker login -u '${DOCKER_USER}' --password-stdin"

                        echo "Construyendo imagen optimizada AMD64..."
                        sh "docker buildx build --platform linux/amd64 -t ${REGISTRY_USER}/${IMAGE_NAME}:${TAG} -t ${REGISTRY_USER}/${IMAGE_NAME}:latest --push ."
                    }
                }
            }
        }
    } 
}