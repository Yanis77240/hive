pipeline {
    agent { 
        node {
            label 'docker-tdp-builder'
            }
      }
    environment {
        number="${currentBuild.number}"
      }
    triggers {
        pollSCM '0 1 * * *'
      }
    stages {       
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
                mvn clean install -Pdist -Phadoop-2 -DskipTests
                '''
            }
        }
        /*stage('Test') {
            steps {
                echo "Testing..."
                sh '''
                mvn test -Phadoop-2 --fail-never
                '''
            }
        }*/        
        stage("Publish to Nexus Repository Manager") {
            steps {
                echo "Deploy..."
                withCredentials([usernamePassword(credentialsId: '4b87bd68-ad4c-11ed-afa1-0242ac120002', passwordVariable: 'pass', usernameVariable: 'user')]) {
                    sh 'mvn deploy -Phadoop-2 -DskipTests -s settings.xml'
                }
            }        
        }
        stage("Publish tar.gz to Nexus") {
            steps {
                echo "Deploy..."
                withCredentials([usernamePassword(credentialsId: '4b87bd68-ad4c-11ed-afa1-0242ac120002', passwordVariable: 'pass', usernameVariable: 'user')]) {
                    sh '''
                    curl -v -u $user:$pass --packaging/target/apache-hive-1.2.3-TDP-0.1.0-SNAPSHOT-bin.tar.gz http://172.19.0.2:8081/repository/maven-tar-files/hive-1.2/apache-hive-1.2.3-TDP-0.1.0-SNAPSHOT-bin-${number}.tar.gz
                    curl -v -u $user:$pass --packaging/target/apache-hive-1.2.3-TDP-0.1.0-SNAPSHOT-src.tar.gz http://172.19.0.2:8081/repository/maven-tar-files/hive-1.2/apache-hive-1.2.3-TDP-0.1.0-SNAPSHOT-src-${number}.tar.gz
                    '''
                }
            }        
        }
    }
}