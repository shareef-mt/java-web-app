pipeline {
    environment {
        VERSION = "latest"
        PROJECT = "683294139580.dkr.ecr.us-east-1.amazonaws.com/sample-java"
        IMAGE = "$PROJECT:$VERSION"
        ECRURL = "https://683294139580.dkr.ecr.us-east-1.amazonaws.com/sample-java"
        ECRCRED = "ecr:us-east-1:aws_credentials"
    }
       
    agent any
    tools {
        maven 'maven'
	terraform 'terraform'
    }
	
    stages {
        stage ('Initialize') {
            steps {
                sh '''
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}/bin/mvn"
                '''
            }
        }
       
        stage('SCM Checkout') {
            steps {
            // Get source code from Gitlab repository
                checkout([$class: 'GitSCM', branches: [[name: '*/develop']], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: '']], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'github_cred', url: 'https://github.com/shareef-mt/java-web-app.git']]])
            }
        }
       
        stage('Mvn Package') {
            steps {
                sh 'mvn -B -DskipTests clean package -e'
            }
        }	
        stage('Docker Image Build') {
            steps {
            sh '''
                    pwd
                    echo "PATH = ${PATH}"
                    echo "PATH = ${IMAGE}"
                '''
                script {
                    sh 'docker version'
			docker.build('$IMAGE')
		   // sh 'docker build -t 683294139580.dkr.ecr.us-east-1.amazonaws.com/sample-java:${VERSION} .'
                }
            }
        }
	stage('Aws Ecr Repo Creation') {
            steps {
                dir("ecr/") {
                    script {
                        sh 'pwd'
                        sh 'terraform init'
                        sh 'terraform plan'
                        sh 'terraform apply --auto-approve'
                           
                    }
                }
            }
        }	
        stage('Scanning & Pushing Docker Image into Aws Repo') {
            steps {
                script {
                    docker.withRegistry(ECRURL, ECRCRED)
                        {
			    //sh 'aws ecr get-login-password --region us-east-1 | docker login --username shareef.ahamadn242@gmail.com --password-stdin $H@reef@786 683294139580.dkr.ecr.region.amazonaws.com'
                            sh 'aws ecr put-image-scanning-configuration --repository-name sample-java --image-scanning-configuration scanOnPush=true --region us-east-1'
				docker.image(IMAGE).push()
			    //sh 'docker push 683294139580.dkr.ecr.us-east-1.amazonaws.com/sample-java:${VERSION}'
                 
                        }
                }
            }
        }
       
        stage('Deploy Aws Ecr image into Aws Ecs') {
            steps {
                dir("ecs/") {
                    script {
                        sh '''
                            terraform init
                            terraform plan
                            terraform apply --auto-approve
       
                           '''
                    }
                }
            }
        }
       
    }  
    }
