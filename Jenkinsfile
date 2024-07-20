def customimage
def TAG
def jobType='JOBTYPE'
def ECRACCOUNT='731760562474.dkr.ecr.ap-south-1.amazonaws.com/'


pipeline{
        agent {
          label 'slave'
        }
        environment {
        AWS_PROFILE="eks-uat-k8s"
        KUBECONFIG="$HOME/.kube/dev-eks-v124.conf"
        ENVIRONMENT= "JENKINSENV"
    }
    stages{
        stage('Clone Stage') {
         steps {
                    
            dir ('automation/code') {
              git branch: '$BRANCH_NAME',
              credentialsId: 'bitbucketcred',
              url: 'https://build-jenkins01@bitbucket.org/godigit/sourcecoderepo.git'
             }
   }
}

        stage('Build Stage') {
          when {
            expression {
              jobType == "JAVA"
            }
          }
         steps {
	    dir ('automation/code') {
            script{
                try{
		          sh 'mvn clean package'
            }
            catch (exc) {
                echo 'Maven Build Failed'
            throw exc
             } 
		}
   }
}
}

                stage('ECR Authentication'){
                  steps{
                          
                      script{
                          try{

                           echo "authentication with ECR"
                        sh 'aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin '+ECRACCOUNT+''
                        
                        sh "aws ecr describe-repositories --repository-names dev-ECR_repo --region ap-south-1 || aws ecr create-repository --repository-name dev-ECR_repo --region ap-south-1"
                     //   sh '$(aws ecr get-login --no-include-email --region ap-south-1)'
                        //sh 'aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin ${ECRACCOUNT}ECR'
                 }
                 catch (exc) {
            echo 'ECR Authentication Failed'
            throw exc
            }

                        }
                  }
                }

                stage('Docker Image Creation'){
                  steps {
                          
		    dir ('automation') {
                    script{
           echo 'Starting to build docker image'
            try{
                           TAG = "${BUILD_TIMESTAMP}-V${BUILD_NUMBER}"
               customimage = docker.build("${ECRACCOUNT}dev-ECR_repo:${TAG}", "--network host --pull -f Dockerfile.dev .")
                    }
                   catch (exc) {
            echo 'Docker Image Creation Failed'
            throw exc
            }

                        }
           }
                  }
                }

       

                stage('Push Docker Image'){
                steps{
                        
                 script{
                     try{
                        customimage.push()
                 }
                  catch (exc) {
            echo 'Docker Image push to ECR failed'
            throw exc

                        }
                 }
                 }
     }
  
  

                 stage('Deployment') {
            steps {
                    
		dir ('automation') {
                 script {
                     try{
                    sh "sed -i  's/TAG/${TAG}/g' helm-automation/values-${ENVIRONMENT}.yaml"
                    sh '''sed  -i  "s/BUILD_NUMBER/\\"v${BUILD_NUMBER}\\"/g" helm-automation/Chart.yaml '''
                    sh "helm upgrade --install dev-automation --create-namespace -n automation -f helm-automation/values-${ENVIRONMENT}.yaml ./helm-automation/"
                      }
                    catch (exc) {
                    echo "deployment failed"
                    throw exc
                  }
                 }
    }
    }
    }

          stage('Remove the Docker Image'){
                    steps{
                            
                        script{
                           try{
                               echo"Docker remove "
                               sh  "docker rmi  -f ${ECRACCOUNT}dev-ECR_repo:${TAG}"

                           }
                           catch (exc) {
                    echo 'Failed to remove docker image'
                    throw exc
                           }
                        }
                    }
                    }    


              stage('CleanWorkspace') {
            steps {
             
                cleanWs()
                 }
                }
 }
 }


