pipeline {
    agent any

    parameters {
        string(name: 'DOCKER_TAG', defaultValue: 'latest', description: 'Docker Tag')
    }

    tools {
        maven 'maven3'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Git Checkout') {
            steps {
                git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/Basudevjena/Multi-Tier-BankApp-CI.git'
            }
        }

        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }

        stage('Unit Test') {
            steps {
                sh "mvn test -DskipTests=true"
            }
        }

        stage('Trivy File System Scan') {
            steps {
                sh "trivy fs --format table -o fs.html ."
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=bankapp -Dsonar.projectName=bankapp -Dsonar.java.binaries=target"
                }
            }
        }
        stage('Build') {
            steps {
                sh "mvn package -DskipTests=true"
            }
        }

        stage('Publish the artifacts to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-settings', jdk: '', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy -DskipTests=true"
                }
            }
        }

        stage('Build and Tag docker image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh "docker build -t basudevjena1/bankapp:${params.DOCKER_TAG} ."
                    }
                }
            }
        }

        stage('Trivy Docker Image Scan') {
            steps {
                sh "trivy image --format table -o dimage.html basudevjena1/bankapp:${params.DOCKER_TAG}"
            }
        }

        stage('Push the docker image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh "docker push basudevjena1/bankapp:${params.DOCKER_TAG}"
                    }
                }
            }
        }
          stage('Update yml manifests in other repo') {
    steps {
        script {
            withCredentials([gitUsernamePassword(credentialsId: 'git-cred', gitToolName: 'Default')]) {
                sh '''
                    
                    
     git clone https://github.com/Basudevjena/Multi-Tier-BankApp-CD.git
     cd Multi-Tier-BankApp-CD
     ls -l bankapp
     repo_dir=$(pwd)
     sed -i "s|image: basudevjena1/bankapp:.*|image: basudevjena1/bankapp:"${DOCKER_TAG}"|" ${repo_dir}/bankapp/bankapp-ds.yml     
   '''
   
sh '''
     echo "Updated yml file contents:"
     cat Multi-Tier-BankApp-CD/bankapp/bankapp-ds.yml
   '''
sh '''
   cd Multi-Tier-BankApp-CD
   git config user.email "basudevjenaeeee@gmail.com"
   git config user.name "Basudevjena"
   '''
   
sh '''
   cd Multi-Tier-BankApp-CD
   ls
   git add bankapp/bankapp-ds.yml
   git commit -m "Updated image tag to ${DOCKER_TAG}"
   git push origin main
   '''
            }
        }
    }
 }
 }
}
