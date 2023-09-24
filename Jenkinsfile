    pipeline{
        agent any
        tools {
            jdk 'jdk17'
            maven 'maven3'
            
        }
        environment {
            SCANNER_HOME=tool 'sonar-scanner'
        }
        stages{
            stage ('clean Workspace'){
                steps{
                    cleanWs()
                }
            }
            stage ('checkout scm') {
                steps {
                    git 'https://github.com/PARPSY1122/jpetstore-6.git'
                }
            }
            stage ('maven compile') {
                steps {
                    sh 'mvn clean compile'
                }
            }
            stage ('maven Test') {
                steps {
                    sh 'mvn test'
                }
            }
            stage("Sonarqube Analysis "){
                steps{
                    withSonarQubeEnv('sonar-server') {
                        sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=poststore1 \
                        -Dsonar.java.binaries=. \
                        -Dsonar.projectKey=poststore1 '''
                    }
                }
            }
            stage("quality gate"){
                steps {
                    script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token' 
                    }
                }
            }
            stage ('Build war file'){
                steps{
                    sh 'mvn clean install -DskipTests=true'
                }
            }
            stage("OWASP Dependency Check"){
                steps{
                     dependencyCheck additionalArguments: '--scan ./ --format XML ', odcInstallation: 'dp-check'
                    dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
                }
            }
            
            stage('Install Docker') {
                steps {
                    dir('Ansible'){
                        script {
                            ansiblePlaybook credentialsId: 'ssh', disableHostKeyChecking: true, installation: 'ansible', inventory: '/etc/ansible/', playbook: 'docker-playbook.yaml'
                        }     
                   }    
                }
            }


        }
    }
