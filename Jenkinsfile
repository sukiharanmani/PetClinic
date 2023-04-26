pipeline {
    
    agent {
        label 'agent_1'
    }
    
    tools {
          maven 'MAVEN3'
          jdk 'JDK8'
    }

    options {
        buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '', numToKeepStr: '10')
        disableConcurrentBuilds()
        timestamps()
    }
    
    stages {
        stage('Code Checkout'){
            steps {
                deleteDir()
                echo "code checkout"
                git credentialsId: 'github-creds', url: "https://github.com/gkdevopsPetClinic.git"
            }
        }
        
        stage('Code Build'){
            steps {
                sh "mvn test"
            }
        }

        stage('SonarQube Scan'){
          environment {
            SCANNER_HOME = tool 'sonar_scanner'
          }
          steps {
            withSonarQubeEnv (installationName: 'SonarQube') {
              sh "${SCANNER_HOME}/bin/sonar-scanner -Dproject.settings=sonar-project.properties"
            }
          }
        }
        stage("SonarQube Quality Gate") {
            steps {
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        /*
        stage('SCA Test'){
            steps {
                sh "snyk test --severity-threshold=critical"
            }
        }
        */
        stage('Maven Package'){
            steps {
                sh "mvn -Dtests.skip=false package"
            }
        }
        stage('Docker Build'){
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-credentials', passwordVariable: 'dockerpassword', usernameVariable: 'dockerusername')]) {
                  sh "sudo docker login -u $dockerusername -p $dockerpassword"
                }
                sh '''
                sudo docker image build -t chgoutam/petclinic:$BUILD_ID .
                sudo docker image push chgoutam/petclinic:$BUILD_ID
                '''
            }
        }
        stage('Docker Image Scan'){
            steps {
                sh "trivy image chgoutam/petclinic:$BUILD_ID"
            }
        }
    }
}
