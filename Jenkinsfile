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
      stage ('Upload file to artifactory') {
            steps {
                rtUpload (
                    // Obtain an Artifactory server instance, defined in Jenkins --> Manage Jenkins --> Configure System:
                    serverId: jfrog,
                    spec: """{
                            "files": [
                                    {
                                        "pattern": "**/target/*.war",
                                        "target": "libs-release-local"
                                    }
                                ]
                          }"""
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
