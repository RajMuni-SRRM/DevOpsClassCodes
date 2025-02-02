
pipeline{
    tools{
        jdk 'myjava'
        maven 'mymaven'
    }
    environment {
       // IMAGE_NAME = '20636/addreebook'
        registry = "20636/addreebook"
        registryCredential = 'dockerhubcred'
        dockerImage = ''
    }
	agent any
      stages{
           stage('Checkout'){
	    
               steps{
		 echo 'cloning..'
                 git 'https://github.com/RajMuni-SRRM/DevOpsClassCodes.git'
              }
          }
          stage('Compile'){
             
              steps{
                  echo 'compiling..'
                  sh 'mvn compile'
	      }
          }
          stage('CodeReview'){
		  
              steps{
		    
		  echo 'codeReview'
                  sh 'mvn pmd:pmd'
              }
          }
           stage('UnitTest'){
		  
              steps{
	         echo 'Testing'
                  sh 'mvn test'
              }
               post {
               success {
                   junit 'target/surefire-reports/*.xml'
               }
           }	
	   }
           stage('MetricCheck'){

              steps{
                  sh 'mvn cobertura:cobertura -Dcobertura.report.format=xml'
              }

          }
          stage('Package'){
		  
              steps{
		  
                  sh 'mvn package'
              }
          }
	  //stage('Build Docker Image'){
	   //   steps{
		//      sh 'docker build -t ${IMAGE_NAME}:V${BUILD_NUMBER} .'
		  //    sh 'docker push ${IMAGE_NAME}:V${BUILD_NUMBER}'
	      //}
	  //}

      //}
      //post {
	//  success{
	  //    sh 'docker rmi ${IMAGE_NAME}:V${BUILD_NUMBER}'
	    // }
    stage('Building image') {
      steps{
        script {
          dockerImage = docker.build registry + ":$BUILD_NUMBER"
        }
      }
    }
    stage('Push Image to docker hub') {
      steps{
         script {
            docker.withRegistry( '', registryCredential ) {
            dockerImage.push()
          }
        }
      }
    }
    stage('Remove Unused docker image') {
      steps{
        sh "docker rmi $registry:$BUILD_NUMBER"
      }
    }
    stage('Deploy docker image in DEV machine using ansible'){
	    steps{
		    ansiblePlaybook credentialsId: 'ec2user',disableHostKeyChecking: true, extras: '--extra-vars "imageversion=${BUILD_NUMBER}"', installation: 'ansible', inventory: 'dev-inv', playbook: 'ansible-deploy.yml'
	    }
    }
   }
}
