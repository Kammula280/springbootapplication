pipeline {
    agent {
        label 'master'
    }
    tools {
        maven 'Maven3.6.3'
   }
   options{
       timestamps()
        timeout(time:60,unit:'MINUTES')
        buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '5', daysToKeepStr: '', numToKeepStr: '5'))
        disableConcurrentBuilds()
    }
   
   triggers{
       pollSCM('* * * * *')
   }
   environment{
    BRANCH_NAME= "${params.branch}"
   }
   parameters{
    string (name : 'version', defaultValue:'1.0', description : 'enter the version')
    choice (name : 'branch', choices :['dev', 'master', 'test'], description : 'select your branch')
    string (name : 'Clustername', defaultValue:'EKS_Cluster', description : 'enter the Clustername')
    choice (name : 'region', choices :['ap-south-1', 'us-east-1', 'us-east-2'], description : 'select your region')
   }

   stages {
       stage ('display env and checkout code'){
           steps{
           parallel (
               'display env': {
                   echo " Environment being implemented is ${BRANCH_NAME}"
               },
               'CheckOutCode':{
                   git branch: 'dev', credentialsId: 'githubcred', url: 'https://github.com/Kammula280/springbootapplication.git'

               }
            
           )
           }
         
       }
       stage('Build Artifact'){
           steps{
           sh 'mvn clean package'
           }
       }
       stage ('Execute Sonarqube report'){
            steps{
                sh 'mvn sonar:sonar'
            }
            }
       stage('upload artifact'){
            steps {
                sh 'mvn deploy'
            }
            }
       stage('Build docker image'){
           steps{
       sh "docker build -t kammula280/springbootapplication ."
           }
       }
       
       
       stage ('Push Docker Image to Repository'){
           steps{
               withCredentials([string(credentialsId: 'docker_hub_credentials', variable: 'docker_hub_credentials')]) {
                   sh "docker login -u kammula280 -p ${docker_hub_credentials}"
                   sh "docker push kammula280/springbootapplication:latest"
                   }
       }
       }
        stage('Get Kubeconfig'){
           steps{
                withAWS(credentials: 'kuberentes', region: 'ap-south-1') {
                    sh "aws eks update-kubeconfig --name ${params.Clustername} --region ${params.region}"}
               
           }
       }
        stage('Deploy to kuberentes'){
           steps{
               withAWS(credentials: 'kuberentes', region: 'ap-south-1') {
               sh "kubectl apply -f springBootMongo.yml"
               }

           }
       }
}
post{
        always{
                mail bcc: '', body: 'Your Jenkins Build is always', cc: '', from: '', replyTo: '', subject: 'Build is Success', to: 'kammularamgopal@gmail.com'
            cleanWs()
            }
            success{
                mail bcc: '', body: 'Your Jenkins Build is Success', cc: '', from: '', replyTo: '', subject: 'Build is Success', to: 'kammularamgopal@gmail.com'
            cleanWs()
            }
            failure{
                mail bcc: '', body: 'Your Jenkins Build is Failed', cc: '', from: '', replyTo: '', subject: 'Build is Failed', to: 'kammularamgopal@gmail.com'
            cleanWs()
            }
        }
}
