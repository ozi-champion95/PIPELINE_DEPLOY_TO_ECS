
	pipeline {
			agent any
			environment {
			   registryCredential = 'ecr:us-east-1:awscreds'
			   appRegistry = "611913014199.dkr.ecr.us-east-1.amazonaws.com/vprofileappimg"
			   vprofileRegistry = "https://611913014199.dkr.ecr.us-east-1.amazonaws.com"
			   cluster = "vprofile"
			   service = "vprofileappsvc"
			}
			

			stages {
			   stage('Fetc Code') {
			      steps {
			         git branch: 'docker', url: 'https://github.com/hkhcoder/vprofile-project.git'
			      }
			   }

			   

			   stage('TEST'){
			      steps {
			         sh 'mvn test'
			      }
			   }

			   stage('Code Analysis with Checkstyle'){
			      steps {
			         sh 'mvn checkstyle:checkstyle'
			      }
			      post {
			         success {
			            echo 'Generated Analysis Result'
			         }
			      }
			   }

			   stage('build && SonarQube Analysis') {
			      environment {
			         scannerHome = tool 'sonar4.7'
			      }
			      steps {
			         withSonarQubeEnv('sonar') {
			            sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
		                   -Dsonar.projectName=vprofile-repo \
		                   -Dsonar.projectVersion=1.0 \
		                   -Dsonar.sources=src/ \
		                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
		                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
		                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
		                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
			         }

			      }
			   }

			   stage('Quality Gate') {
			      steps {
			         timeout(time: 1, unit: 'HOURS') {
			            // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
			            // true = set pipeline to UNSTABLE, false = don't
			            waitForQualityGate abortPipeline: true
			         }
			      }
			   }

			   stage('Build App Image') {
			      steps {
			         script {
			                dockerImage = docker.build(appRegistry + ":$BUILD_NUMBER", "./Docker-files/app/multistage/")
			         }
			      }
			   }

			   stage("Upload App Image") {
			      steps{
			       script {
			         docker.withRegistry( vprofileRegistry, registryCredential ) {
			            dockerImage.push("$BUILD_NUMBER")
			            dockerImage.push('latest')
			         }
			      }
			   }
			}

			stage('Deploy To ECS') {
			      steps {
			     withAWS(credentials: 'awscreds', region: 'us-east-1') {
			        sh 'aws ecs update-service --cluster ${cluster} --service ${service} --force-new-deployment'
			     }
			      
			     }
			   }

			   

			   }

			   post {
			   always {
			      echo 'Slack Notification.'
			      slackSend channel: '#devopscicdudemy',
			         color: COLOR_MAP[currentBuild.currentResult],
			         message: "*${currentBuild.currentResult}:* job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
			   }
			}

		}