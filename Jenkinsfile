pipeline{
    agent{
         node{
             label "Slave-1"
             customWorkspace "/home/jenkins/bankapp"
         }
    }
    environment{
        JAVA_HOME="/usr/lib/jvm/java-17-amazon-corretto.x86_64"
        PATH="$PATH:$JAVA_HOME/bin:/opt/apache-maven/bin:/opt/node-v16.0.0/bin:/usr/local/bin"
        ARGOCD_PASSWORD=credentials('ARGOCD_PASSWORD')
    }
    parameters { 
        string(name: 'REPO_NAME', defaultValue: '', description: 'Provide the ECR Repository Name for Application Image')
        string(name: 'TAG_NAME', defaultValue: '', description: 'Provide the TAG Name')
    }
    stages{
        stage("Clone-Code"){
            steps{
                cleanWs()
                checkout scmGit(branches: [[name: 'main']], extensions: [], userRemoteConfigs: [[credentialsId: 'github-cred', url: 'https://github.com/singhritesh85/Bank-App-multibranch.git']])
            }
        }
        stage("Build"){
            steps{
                sh 'mvn clean install'
            }
        }
        stage("Docker Build"){
            steps{
                sh 'docker system prune -f --all'
                sh 'docker build -t myimage:1.01 -f Dockerfile-Project-1 .'
                sh 'docker tag myimage:1.01 ${REPO_NAME}:${TAG_NAME}'
                sh 'aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin 027330342406.dkr.ecr.us-east-2.amazonaws.com'
                sh 'docker push ${REPO_NAME}:${TAG_NAME}'
            }
        }
        stage("Deployment"){
            steps{
                sh 'argocd login argocd.singhritesh85.com --username admin --password $ARGOCD_PASSWORD --skip-test-tls  --grpc-web'
                sh 'argocd app create bankapp --project default --repo https://github.com/singhritesh85/helm-repo-for-ArgoCD.git --path ./folo --dest-namespace bankapp --dest-server https://kubernetes.default.svc --helm-set service.port=80 --helm-set image.repository=${REPO_NAME} --helm-set image.tag=${TAG_NAME} --helm-set replicaCount=1 --upsert'
                sh 'argocd app sync bankapp'
            }
        }
    }
    post {
        always {
            mail bcc: '', body: "A Jenkins Job with Job Name ${env.JOB_NAME} has been executed", cc: '', from: '', replyTo: '', subject: "Jenkins Job ${env.JOB_NAME} has been executed", to: 'abc@gmail.com'
        }
        success {
            mail bcc: '', body: "A Jenkins Job with Job Name ${env.JOB_NAME} and Build Number=${env.BUILD_NUMBER} has been executed Successfully, Please Open the URL ${env.BUILD_URL} and click on Console Output to see the Log. The Result of execution is ${currentBuild.currentResult}", cc: '', from: '', replyTo: '', subject: "Jenkins Job ${env.JOB_NAME} has been Sucessfully Executed", to: 'abc@gmail.com'
        }
        failure {
            mail bcc: '', body: "A Jenkins Job with Job Name ${env.JOB_NAME} and Build Number=${env.BUILD_NUMBER} has been Failed, Please Open the URL ${env.BUILD_URL} and click on Console Output to see the Log. The Result of execution is ${currentBuild.currentResult}", cc: '', from: '', replyTo: '', subject: "Jenkins Job ${env.JOB_NAME} has been Failed", to: 'abc@gmail.com'
        }
    }
}
