pipeline {
  agent any
  stages {
    stage('workflow:sandbox') {
      when { anyOf { environment name: 'DATAGOV_WORKFLOW', value: 'sandbox' } }
      stages {
        stage('build') {
          steps {
            ansiColor('xterm') {
              sh 'bin/jenkins-deploy build'
              zip zipFile: 'datagov-deploy.zip', archive: true
            }
          }
        }
        stage('deploy:sandbox') {
          when { anyOf { branch 'develop'; branch 'bugfix/jenkins-branch' } }
          environment {
            ANSIBLE_VAULT_FILE = credentials('ansible-vault-secret')
            SSH_KEY_FILE = credentials('datagov-sandbox')
          }
          steps {
            ansiColor('xterm') {
              sh 'bin/jenkins-deploy ping sandbox'
            }
            ansiColor('xterm') {
              sh 'echo bin/jenkins-deploy deploy sandbox site.yml'
            }
          }
        }
      }
    }
    stage('workflow:production') {
      when { anyOf { environment name: 'DATAGOV_WORKFLOW', value: 'production' } }
      stages {
        stage('build') {
          steps {
            ansiColor('xterm') {
              sh 'bin/jenkins-deploy build'
              zip zipFile: 'datagov-deploy.zip', archive: true
            }
          }
        }
        stage('deploy:staging') {
          when {
            anyOf {
              branch 'release/*'
              branch 'master'
              branch 'feature/jenkins-artifact'
            }
          }
          environment {
            ANSIBLE_VAULT_FILE = credentials('ansible-vault-secret')
            SSH_KEY_FILE = credentials('datagov-stage-ssh')
          }
          steps {
            ansiColor('xterm') {
              sh 'bin/jenkins-deploy ping staging'
            }
            ansiColor('xterm') {
              sh 'bin/jenkins-deploy deploy staging site.yml'
            }
          }
        }
        stage('deploy:mgmt') {
          when {
            anyOf {
              branch 'master'
            }
          }
          environment {
            ANSIBLE_VAULT_FILE = credentials('ansible-vault-secret')
            SSH_KEY_FILE = credentials('datagov-mgmt-ssh')
          }
          steps {
            ansiColor('xterm') {
              sh 'bin/jenkins-deploy ping mgmt'
            }
            ansiColor('xterm') {
              sh 'bin/jenkins-deploy deploy mgmt site.yml'
            }
          }
        }
        stage('deploy:production') {
          when {
            anyOf {
              branch 'master'
            }
          }
          environment {
            ANSIBLE_VAULT_FILE = credentials('ansible-vault-secret')
            SSH_KEY_FILE = credentials('datagov-prod-ssh')
          }
          steps {
            ansiColor('xterm') {
              sh 'bin/jenkins-deploy ping production'
            }
            ansiColor('xterm') {
              sh 'bin/jenkins-deploy deploy production site.yml'
            }
          }
        }
      }
    }
  }
  post {
    always {
      step([$class: 'GitHubIssueNotifier', issueAppend: true])
      cleanWs()
    }
  }
}
