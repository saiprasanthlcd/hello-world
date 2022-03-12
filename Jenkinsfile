pipeline {
  agent any
  tools {
    maven 'maven-3.8.3'
    jdk 'Java11'
  } 
  stages{
      stage('Delete Workspace') {
        steps{
            cleanWs()
        }
      }
      stage('Git Clone') {
        steps{
            git 'https://github.com/saiprasanthlcd/hello-world.git'
        }
      }
      stage('compile code') {
        steps {
          sh 'mvn compile'
        }
      }
      stage('SonarQube Analysis') {
        steps {
          // sonarqube plugin needs to be installed and should be configured in configure system in Jenkins
          withSonarQubeEnv('sonarqube') {
            sh 'mvn sonar:sonar'
          }
        }
      }
      stage('Build Package') {
        steps{
            sh 'mvn clean install'
        }
      }
      stage ('Server Connect to JFROG Artifactory'){
          steps {
              rtServer (
                id: "jfrog",
                url: 'https://jfrog.sidhudevops.co.in/artifactory/',
                username: 'jenkins',
                password: '9849199983@Dad',
                bypassProxy: true,
                  timeout: 300
                      )
          }
      }
      stage('Upload to JFROG'){
          steps{
              rtUpload (
                serverId:"jfrog" ,
                spec: '''{
                  "files": [
                    {
                    "pattern": "**/*.war",
                    "target": "libs-snapshot-local"
                    }
                          ]
                          }''',
                      )
          }
      }
      stage ('Publish build info') {
          steps {
              rtPublishBuildInfo (
                  serverId: "jfrog"
              )
          }
      }
      stage('Copy Artifacts to Ansible Server over SSH') {
        steps{
            sshPublisher(publishers: [sshPublisherDesc(configName: 'ansible', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: 'cd /opt/docker && ansible-playbook -i hosts -vv createimgdocker.yaml;', execTimeout: 0, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '//opt//docker', remoteDirectorySDF: false, removePrefix: 'webapp/target', sourceFiles: 'webapp/target/*.war')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: true)])
        }
      }
      stage('Deploy in Kubernetes Server') {
        input {
          message "Please select YES or NO to proceed with Deployment"
          ok "Yes, We can Proceed"
          submitter "SaiPrasanth"
          parameters {
            string(name: 'PERSON', defaultValue: 'SaiPrasanth', description: 'Please provide name of person who is approving')
          }
        }
        steps{
          sshPublisher(publishers: [sshPublisherDesc(configName: 'ansible', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: 'cd /opt/docker && ansible-playbook -i hosts -vv kube_deploy.yaml;', execTimeout: 0, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '', remoteDirectorySDF: false, removePrefix: '', sourceFiles: '')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
        }
      }
  }
}
