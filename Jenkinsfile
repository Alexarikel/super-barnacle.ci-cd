pipeline {
    agent { label 'aws && docker' }
    triggers {
        pollSCM '* * * * *'
    }
    environment {
    AWS_ACCOUNT_ID = "exampleid"
    AWS_DEFAULT_REGION = "us-east-1"
    AWS_ACCESS = credentials('aws_access')
    IMAGE_REPO_NAME = "my-app"
    IMAGE_TAG = "1.${BUILD_NUMBER}"
    REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
    MAIL = "alexarikel@gmail.com"
    }
    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM', 
                branches: [[name: '*/main']],
                extensions: [
                [$class: 'SparseCheckoutPaths', 
                sparseCheckoutPaths:[[$class:'SparseCheckoutPath', path:'/image/']]]
                ],
                userRemoteConfigs: [[credentialsId: 'github_repo', url: 'git@github.com:Alexarikel/super-barnacle.git/']]])
            }
        }
        stage('Logging into AWS ECR') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'aws_access', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    echo "Logging into AWS ECR.."
                    sh """
                    aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com
                    """
                }
            }
        }
        stage('Build app image') {
            steps {
                echo "Building image.."
                sh """
                cd image/
                docker build -t app:1.${BUILD_NUMBER} -t app:latest .
                """
            }
        }
        stage('Test app image') {
            steps {
                echo "Testing image.."
                sh '''
                res="$(docker images | grep app | grep "1.${BUILD_NUMBER}" -c)"
                if [ $res -ne 1 ]; then echo failed test && exit 1; fi
                '''
            }
        }
        stage('Push image to ECR repository') {
	        steps {
                echo "Pushing image to a repo.."
                sh """
                docker tag app:${IMAGE_TAG} ${REPOSITORY_URI}:$IMAGE_TAG
                docker push $REPOSITORY_URI:${IMAGE_TAG}
                """
            }
        }
        stage('Change application version') {
            steps {
                sh """
                git clone git@github.com:Alexarikel/super-barnacle.ci-cd.git
                cd super-barnacle.ci-cd/app-chart/
                sed -i 's/appVersion:.*/appVersion: "${IMAGE_TAG}"/' Chart.yaml
                git config user.email "${MAIL}" && git config user.name "Jenkins"
                git checkout main && git add Chart.yaml && git commit -m "bump version - ${IMAGE_TAG}" && git push
                rm -r ../../super-barnacle.ci-cd/
                """
            }
        }
    }
    post {
        failure {
            mail to: "${MAIL}",
                subject: "Failed Pipeline: ${currentBuild.fullDisplayName}",
                body: "Something is wrong with ${env.BUILD_URL}"
        }
    }
}


