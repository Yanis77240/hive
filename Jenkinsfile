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
        stage('clone') {
            steps {
                echo "Cloning..."
                git branch: 'branch-3.1-TDP-alliage', url: 'https://github.com/Yanis77240/hive'
            }
        }
        stage('Build') {
            steps {
                echo "Building..."
                sh '''
                mvn clean install -Pdist -DskipTests
                '''
            }
        }
        stage("Publish to Nexus Repository Manager") {
            steps {
                echo "Deploy..."
                withCredentials([usernamePassword(credentialsId: '4b87bd68-ad4c-11ed-afa1-0242ac120002', passwordVariable: 'pass', usernameVariable: 'user')]) {
                    sh 'mvn clean deploy -Pdist -DskipTests -s settings.xml'
                }
            }        
        }
    }
}