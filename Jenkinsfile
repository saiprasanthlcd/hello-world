pipeline {
  agent any
  tools {
    maven 'maven-3.8.3'
    jdk 'Java11'
    Git 'Git'
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
      stage('Junit test') {
        steps{
          sh 'mvn test'
        }
        post{
            always {
              junit '**/target/surefire-reports/*.xml'  
            }
        }
      }
      stage('Metrics Check') {
        steps {
          // cobertura plugin should be installed
          sh 'mvn cobertura:cobertura -Dcobertura.report.format=xml'
        }
        post {
          always {
            cobertura autoUpdateHealth: false,
             autoUpdateStability: false,
              coberturaReportFile: '**/target/site/cobertura/coverage.xml',
               conditionalCoverageTargets: '70, 0, 0',
                failUnhealthy: false,
                 failUnstable: false,
                  lineCoverageTargets: '80, 0, 0',
                   maxNumberOfBuilds: 0,
                    methodCoverageTargets: '80, 0, 0',
                     onlyStable: false,
                      sourceEncoding: 'ASCII',
                       zoomCoverageChart: false
          }
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
