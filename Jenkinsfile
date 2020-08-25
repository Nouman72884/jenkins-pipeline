// @Library('github.com/releaseworks/jenkinslib') _
instance_id=''
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
                         //sh 'instance_id=$(aws ec2 describe-instances --filters Name=tag:Name,Values=nouman-ec2 --query Reservations[0].Instances[0].InstanceId --region=us-east-1 --output text)'
                         instance_id=sh(
                               script: 'aws ec2 describe-instances --filters Name=tag:Name,Values=nouman-ec2 --query Reservations[0].Instances[0].InstanceId --region=us-east-1 --output text',
                               returnStdout: true,
                          )
               } 
             }
             stage('deregistering instances') {
                    steps {
                          sh 'aws elb deregister-instances-from-load-balancer --load-balancer-name nouman-classic-lb --instances ${instance_id} --region=us-east-1'
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
                           execCommand: 'sudo apt-get update -y:sudo apt-get install python-pip -y;cd /tmp/tmp/;pip install *.whl;sudo apt install python3-flask -y;sudo apt-get install at;sudo systemctl start atd;sudo systemctl enable atd;cd 01-hello-world/;export FLASK_APP=hello.py;killall flask;echo "flask run --host=0.0.0.0" | at -m now',
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
                          sh 'instance_id=$(aws ec2 describe-instances --filters Name=tag:Name,Values=nouman-ec2 --query Reservations[0].Instances[0].InstanceId --region=us-east-1 --output text)'
                          sh 'aws elb register-instances-with-load-balancer --load-balancer-name nouman-classic-lb --instances $instance_id --region=us-east-1'
               }  
             }
              }
}
