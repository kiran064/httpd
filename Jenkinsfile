pipeline{
    agent {
        label {
            label 'ansible_controller'
            customWorkspace '/home/ec2-user/project'
        }
    }
    stages {
        stage ('clone') {
            steps {
                cleanWs()
                git 'https://github.com/kiran064/game-of-life.git'
                writeFile file: 'docker-compose.yaml', text: '''version: "3"
services:
  web1:
    image: "tomcat:8.5-jre8-temurin-focal"
    ports:
      - "8080:8080"
    volumes:
      - ./gameoflife-web/target/:/usr/local/tomcat/webapps/ '''

                writeFile file: 'test.yaml', text: '''- 
  name: install docker
  hosts: project
  become: yes
  tasks:
    - name: docker installing
      yum:
        name: docker
        state: present
    - service:
        name: docker
        state: started
    - shell: |
        curl -SL https://github.com/docker/compose/releases/download/v2.15.1/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose
        chmod +x /usr/local/bin
        ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
        chmod +x /usr/bin/docker-compose
        docker-compose down
        docker system prune -af
        rm -rf *
    - copy:
        src: docker-compose.yaml
        dest: docker-compose.yaml
    - shell: |
        docker-compose up -d'''
            }
        }
        stage ("docker_conf") {
            steps {
                sh "mvn clean install"
            }
        }
        stage ("deploy") {
            steps {
                sh "ansible-playbook test.yaml"
            }
        }
    }
}