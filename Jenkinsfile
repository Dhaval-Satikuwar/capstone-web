pipeline {
  agent any
  options { timestamps() }
  environment {
    INV = '/var/lib/jenkins/ansible/hosts.ini'
    ANSIBLE_HOST_KEY_CHECKING = 'False'
  }
  stages {
    stage('Checkout (public)') {
      steps {
        git url: 'https://github.com/Dhaval-Satikuwar/capstone-web.git', branch: 'main'
        sh 'ls -la; ls -la ansible; head -n 30 ansible/deploy.yaml'
      }
    }
    stage('Deploy with Ansible') {
      steps {
        sh 'set -eux; /usr/bin/ansible-playbook -i ${INV} ansible/deploy.yaml'
      }
    }
    stage('Verify (HTTP)') {
      steps {
        sh '''
          set -eux
          APP_IP=$(awk '/^capstone-app-machine/   {for(i=1;i<=NF;i++) if($i ~ /^ansible_host=/){split($i,a,"="); print a[2]}}' ${INV})
          AZ_IP=$(awk  '/^capstone-azure-vm-1/    {for(i=1;i<=NF;i++) if($i ~ /^ansible_host=/){split($i,a,"="); print a[2]}}' ${INV})
          curl -s http://$APP_IP | grep -qi "Welcome to AWS"
          curl -s http://$AZ_IP  | grep -qi "Welcome to Azure"
        '''
      }
    }
  }
  post { always { archiveArtifacts artifacts: 'index-*.html, ansible/**', onlyIfSuccessful: false } }
}
