pipeline {
    agent { 
        node {
            label 'docker-tdp-builder'
            }
      }
    triggers {
        pollSCM '0 1 * * *'
      }
    stages {
        stage ('Invoke pipeline tdp-tez') {
            steps {
                build job: 'tdp-tez'
            }
        }        
        stage('clone') {
            steps {
                echo "Cloning..."
                git branch: 'branch-1.2-TDP-alliage', url: 'https://github.com/Yanis77240/hive'
            }
        }
        stage('Build') {
            steps {
                echo "Building..."
                sh '''
                mvn clean install -Phadoop-2 -DskipTests
                '''
            }
        }
        stage("Publish to Nexus Repository Manager") {
            steps {
                echo "Deploy..."
                withCredentials([usernamePassword(credentialsId: '4b87bd68-ad4c-11ed-afa1-0242ac120002', passwordVariable: 'pass', usernameVariable: 'user')]) {
                    sh 'mvn clean deploy -Phadoop-2 -DskipTests -s settings.xml'
                }
            }        
        }
    }
}