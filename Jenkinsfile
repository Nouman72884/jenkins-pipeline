def SH_WITH_RETRIES_AND_RETURN( String cmd, Integer retries=5, Integer sleepSeconds=10  ){
  def retriesCount = 0
  echo "sh[retries=${retries}]: ${cmd}"
  try {
    retry( retries ){
      retriesCount = retriesCount+1
      echo "attempt ${retriesCount}/$retries"
      if( retriesCount > 1 ){
        sleep sleepSeconds
        sleepSeconds = sleepSeconds+1
      }
      return sh( returnStdout: true, script: "${cmd}").trim()
    }
  } catch(e) {
    throw e
  }
}
pipeline {
    agent any
            stages {    
             stage('SCM checkout') {
                  steps {
                        git url: 'https://github.com/Nouman72884/flask-examples.git'
                        }
             }
             stage('get instance id') {
                   steps {
                         script {
                         instance_id=sh(
                               script:"aws ec2 describe-instances --filters Name=tag:Name,Values=nouman-ec2 --query Reservations[0].Instances[0].InstanceId --region=us-east-1 --output text",
                               returnStdout: true,
                         )
                         echo "Hello ${instance_id}"     
               } 
                   }
             }
             stage('deregistering instances') {
                    steps {
                          echo "${instance_id}"
                          sh "aws elb deregister-instances-from-load-balancer --load-balancer-name nouman-classic-lb --region us-east-1 --instances  ${instance_id} "
                          }
               }
             
             stage('preparation 2') {
                   steps {
                         sh 'pip wheel  -r requirements.txt'
                         }
            }
              stage('transfer artifacts') {
                    steps {
                          sshPublisher(
                          publishers:
                          [sshPublisherDesc
                          (configName: 'webserver',
                           transfers: [sshTransfer(
                           excludes: 'jenkinsfile,pipeline.groovy',
                           execCommand: 'sudo apt-get update -y;sudo apt-get install python-pip -y;cd /tmp/tmp/;pip install *.whl;sudo apt install python3-flask -y;sudo apt-get install at;sudo systemctl start atd;sudo systemctl enable atd;cd 01-hello-world/;export FLASK_APP=hello.py;killall flask;echo "flask run --host=0.0.0.0" | at -m now',
                           execTimeout: 350000,
                           flatten: false,
                           makeEmptyDirs: true,
                           noDefaultExcludes: false,
                           patternSeparator: '[, ]+',
                           remoteDirectory: '/tmp',
                           remoteDirectorySDF: false,
                           removePrefix: '', sourceFiles: '**/*')],
                           usePromotionTimestamp: false,
                           useWorkspaceInPromotion: false,
                           verbose: true)])
                          }
              }
             stage('registering instance') {
                    steps {
                          sh "aws elb register-instances-with-load-balancer --load-balancer-name nouman-classic-lb --region=us-east-1 --instances ${instance_id} "
               }  
             }
              }
}
