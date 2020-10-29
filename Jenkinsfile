pipeline {
  agent none
//check every minute for changes
  triggers {
    pollSCM('*/1 * * * *')
  }
  stages {
    //Build container image
    stage('Build') {
      agent {
        kubernetes {
          label 'jenkinsrun'
          defaultContainer 'dind'
          yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: dind
    image: docker:18.05-dind
    securityContext:
      privileged: true
    volumeMounts:
      - name: dind-storage
        mountPath: /var/lib/docker
  volumes:
    - name: dind-storage
      emptyDir: {}
"""
        }
      }
      steps {
        container('dind') {
          script {
            //we need some extra packages installing
            sh 'apk -Uuv add make groff less python py-pip'
//we need the AWS command line tools
            sh 'pip install awscli'
//we need to authenticate with AWS
            sh '$(aws ecr get-login --no-include-email --region eu-west-1)'
//tell jenkins where to save the image
            //notice this is https:// followed by the repositoryUri you created earlier
            docker.withRegistry('https://AWSACCOUNT.dkr.ecr.eu-west-1.amazonaws.com/app') {
//build the image
              def customImage = docker.build("app:${env.BUILD_ID}")
//upload it to the registry
              customImage.push()
            }
          } //script
        } //container
      } //steps
    } //stage(build)
    //Test goes here
    //SonarQube goes here
    //Documentation generation goes here
    //Deploy goes here
    //Performance testing goes here
  } //stages
} //pipeline
