pipeline {
  agent any

  environment {
    DOCKERHUB_CREDENTIALS = credentials('DOCKER_HUB_CREDENTIAL')
    VERSION = "${env.BUILD_ID}"
     JENKINS_SERVER = "54.174.179.191"
  }

  tools {
    maven "Maven"
  }

  stages {

    stage('Maven Build'){
        steps{
        sh 'mvn clean package  -DskipTests'
        }
    }

     stage('Run Tests') {
      steps {
        sh 'mvn test'
      }
    }

    stage('SonarQube Analysis') {
  steps {
    sh 'mvn clean org.jacoco:jacoco-maven-plugin:prepare-agent install sonar:sonar -Dsonar.host.url=http://${JENKINS_SERVER}:9000/ -Dsonar.login=squ_39a9694cd607275d372753857acf98695db232a1'
  }
}

stage('Check code coverage') {
            steps {
                script {
                    def token = "squ_39a9694cd607275d372753857acf98695db232a1"
                    def sonarQubeUrl = "http://${JENKINS_SERVER}:9000/api"
                    def componentKey = "com.codedecode:order"
                    def coverageThreshold = 0.0

                    def response = sh (
                        script: "curl -H 'Authorization: Bearer ${token}' '${sonarQubeUrl}/measures/component?component=${componentKey}&metricKeys=coverage'",
                        returnStdout: true
                    ).trim()

                    def coverage = sh (
                        script: "echo '${response}' | jq -r '.component.measures[0].value'",
                        returnStdout: true
                    ).trim().toDouble()

                    echo "Coverage: ${coverage}"

                    if (coverage < coverageThreshold) {
                        error "Coverage is below the threshold of ${coverageThreshold}%. Aborting the pipeline."
                    }
                }
            }
        } 

stage('Docker Build and Push') {
      steps {
          sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
          sh 'docker build -t veerendra1976/order-service:${VERSION} .'
          sh 'docker push veerendra1976/order-service:${VERSION}'
      }
    } 

stage('Cleanup Workspace') {
      steps {
        deleteDir()
       
      }
    }

stage('Update Image Tag in GitOps') {
      steps {
         checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[ credentialsId: 'git-ssh', url: 'git@github.com:aashrita2012/deployment-folder.git']])
        script {
       sh '''
          sed -i "s/image:.*/image: veerendra1976\\/order-service:${VERSION}/" aws/order-manifest.yml
        '''
          sh 'git checkout main'
          sh 'git add .'
          sh 'git commit -m "Update image tag"'
        sshagent(['git-ssh'])
            {
                  sh('git push')
            }
        }
      }
    }

  }
}
