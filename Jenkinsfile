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
      stage('Build Package') {
        steps{
            sh 'clean install'
        }
      }
      stage('Copy Artifacts to Ansible Server over SSH') {
        steps{
            sshPublisher(publishers: [sshPublisherDesc(configName: 'ansible', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: 'cd /opt/docker && ansible-playbook -i hosts -vv createimgdocker.yaml;', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '//opt//docker', remoteDirectorySDF: false, removePrefix: 'webapp/target', sourceFiles: 'webapp/target/*.war')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: true)])
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
          sshPublisher(publishers: [sshPublisherDesc(configName: 'ansible', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: 'cd /opt/docker && ansible-playbook -i hosts -vv kube_deploy.yaml;', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '', remoteDirectorySDF: false, removePrefix: '', sourceFiles: '')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
        }
      }
  }
}
