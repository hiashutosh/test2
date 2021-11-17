
def func1() {
  stage ('1st') {
      script {
        echo "stage 1"
        echo "${currentBuild.getBuildCauses()[0].shortDescription}"
        echo "${currentBuild.rawBuild.getCause(hudson.triggers.SCMTrigger$SCMTriggerCause)}"

      
    }
  }
  stage ('2nd') {
      script {
        echo "stage2"
    }
  }
}
def func2() {
  stage ('1st') {
      script {
        echo "stage 1"
        
        echo "${currentBuild.rawBuild.getCause(hudson.triggers.SCMTrigger$SCMTriggerCause)}"
      }
    }
  stage ('2nd') {
      script {
        echo "stage2"
      }
  }
}
pipeline {
  agent {
    kubernetes {
      yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: awscli
    image: amazon/aws-cli:2.3.4
    command: ["/bin/sh"]
    args: ["-c", "while true; do echo hello; sleep 10;done"]
'''
      }
    }
options {
  ansiColor('xterm')
  skipDefaultCheckout true
}

triggers {
  pollSCM '* * * * *'
}

    environment {
        AWS_CONFIG = credentials('ashutosh-aws')
    }
    
    stages {
      stage ('init') {
        steps {
          script {
            sh "mkdir -p Rewardz/"
            checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [[$class: 'ScmName', name: 'test1']], userRemoteConfigs: [[url: 'https://github.com/hiashutosh/test1.git']]])
            checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [[$class: 'ScmName', name: 'test2']], userRemoteConfigs: [[url: 'https://github.com/hiashutosh/test2.git']]])
            // checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'Rewardz']], userRemoteConfigs: [[url: 'https://github.com/hiashutosh/test2.git'], [url: 'https://github.com/hiashutosh/test1.git']]])
            List<String> BOOL = [];
            BOOL = ["func1","func2"];
            env.RicWeb = input  message: 'Want to deploy "ric-web"?',ok : 'Confirm',id :'update_autoscaling',
                            parameters:[choice(choices: BOOL, description: 'Select yes to deploy ric-web to ECR', name: 'RICWEB')] 
            REPO = env.RicAuth?:''
            container('awscli') {
                
            // withCredentials([file(credentialsId: 'ashutosh-aws', variable: 'awsConfig')]) {
                sh '''
                mkdir -p /root/.aws/
                touch /root/.aws/credentials
                cp $AWS_CONFIG /root/.aws/credentials
                yum update -y   && yum install -y jq  && yum clean all
                mkdir -p Rewardz
                '''
            // }
              }
            }
          }
        }
        stage('Setup Docker Pod') {
          steps {
            container('awscli') {
              sh '''/usr/local/bin/aws ec2 describe-instance-status --instance-id i-02553566ece339587 --profile rewardz --region ap-southeast-1 | jq -r '.InstanceStatuses[] | .SystemStatus.Details[] | .Status' | sort -u                '''
            }
          }
        }
        stage ('get func') {
          steps {
            script {
              switch(env.RicWeb) {
                case "func1": 
                  func1();
                  break
                case "func2":
                  func2();
                  break
                default: 
                  echo "default case"
                  break
                }
              }
          }
        }
    }
}