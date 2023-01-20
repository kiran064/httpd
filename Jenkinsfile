pipeline{
    agent {
        label {
            label 'ansible_controller'
            customWorkspace '/home/ansible/project'
        }
    }
    tools { jdk 'java8' }
    stages {
        stage ('clone') {
            steps {
                cleanWs()
                git 'https://github.com/kiran064/index.git'
                sh "git clone https://github.com/kiran064/index.git -b dev"
                writeFile file: 'docker-compose.yaml', text: '''version: "3.9"
services:
  web1:
    image: "httpd"
    ports:
      - "8090:80"
    volumes:
      - ./vertex:/usr/local/apache2/htdocs/
  web2:
    image: "httpd"
    ports:
      - "8080:80"
    volumes:
      - ./tinker:/usr/local/apache2/htdocs/'''
            }
        }
        stage ("docker_conf") {
            steps {
                sh "ansible -m yum -a 'name=docker state=present' project --become"
                sh "ansible -m service -a 'name=docker state=started' project --become"
                /*sh 'ansible -a "curl -SL https://github.com/docker/compose/releases/download/v2.15.1/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose" project --become'
                sh 'ansible -a "chmod +x /usr/local/bin" project --become'
                sh 'ansible -a "ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose" project --become'
                sh 'ansible -a "chmod +x /usr/bin/docker-compose" project --become'*/
            }
        }
        stage ("deploy") {
            steps {
                sh "ansible -a 'docker-compose down' project --become"
                sh "ansible -a 'docker system prune -af' project --become"
                sh "ansible -a 'rm -rf *' project --become"
                sh "ansible -m copy -a 'src=docker-compose.yaml dest=docker-compose.yaml' project --become"
                sh "ansible -m copy -a 'src=./vertex dest=/home/ansible/vertex/ owner=ansible group=ansible mode=0644' qa --become"
                sh "ansible -m copy -a 'src=./index/tinker dest=/home/ansible/tinker owner=ansible group=ansible mode=0644' dev --become"
                sh "ansible -a 'docker-compose up -d web2' dev --become"
                sh "ansible -a 'docker-compose up -d web1' qa --become"
            }
        }
    }
}