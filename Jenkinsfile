// @Library('github.com/releaseworks/jenkinslib') _
pipeline {
    agent any
            stages {    
//              stage("deregistering instances") {
//                    steps{
//                   withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'aws-key', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY']]) {
//                   AWS("--region=us-east-1 elb deregister-instances-from-load-balancer --load-balancer-name nouman-classic-lb --instances i-0633d6cd62185c1b9")
//     }
//              }
//   }
             stage('SCM checkout') {
                  steps {
                        git url: 'https://github.com/Nouman72884/flask-examples.git'
                        }
             }
             stage('deregistering instance') {
                    steps {
                        //withAWS(credentials: 'aws-credentials', region: 'us-east-1') {
                          sh 'instance_id=aws ec2 describe-instances --filters Name=tag:Name,Values=nouman-ec2 --query Reservations[0].Instances[0].InstanceId --output text'
                          sh 'aws elb deregister-instances-from-load-balancer --load-balancer-name nouman-classic-lb --instances $instance_id --region=us-east-1'
                          //}
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
                        //withAWS(credentials: 'aws-credentials', region: 'us-east-1') {
                          sh 'instance_id=$(aws ec2 describe-instances --filters Name=tag:Name,Values=nouman-ec2 --query Reservations[0].Instances[0].InstanceId --output text)'
                          sh 'aws elb register-instances-with-load-balancer --load-balancer-name nouman-classic-lb --instances $instance_id --region=us-east-1'
                          //}
               }  
             }
              }
}
