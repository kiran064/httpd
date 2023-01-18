pipeline{
    agent {
        label {
            label 'ansible_controller'
            customWorkspace '/home/ansible/project'
        }
    }
    stages {
        stage ('clone') {
            steps {
                cleanWs()
                git "https://github.com/kiran064/httpd.git"
                writeFile file: 'docker-compose.yaml', text: '''version: "3.9"
services:
  web1:
    image: "tomcat:9.0.70-jdk11-corretto-al2"
    ports:
      - "8085:8080"
    volumes:
      - ./target:/usr/local/tomcat/webapps
  web2:
    image: "tomcat:9"
    ports:
      - "8090:8080"
    volumes:
      - ./gameoflife-web/target:/usr/local/tomcat/webapps
                '''
            }
        }
        stage ("docker_conf") {
            steps {
                sh "ansible -m yum -a 'name=docker state=present' project --become"
                sh "ansible -m service -a 'name=docker state=started' project --become"
                sh 'ansible -a "curl -SL https://github.com/docker/compose/releases/download/v2.15.1/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose" project --become'
                sh 'ansible -a "chmod +x /usr/local/bin" project --become'
                sh 'ansible -a "ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose" project --become'
                sh 'ansible -a "chmod +x /usr/bin/docker-compose" project --become'
            }
        }
        stage ("deploy") {
            steps {
                sh "ansible -a 'docker-compose up -d' dev --become"
                sh "ansible -a 'docker-compose up -d' qa --become"
            }
        }
    }
}