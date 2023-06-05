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
                git branch: 'branch-2.3-TDP-alliage', url: 'https://github.com/Yanis77240/hive'
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
        stage('Test') {
            steps {
                echo "Testing..."
                sh '''
                mvn test -Dtest=!org.apache.hadoop.hive.ql.TestTxnCommands2WithSplitUpdateAndVectorization -Dtest=!org.apache.hadoop.hive.ql.TestTxnCommands2.testNonAcidToAcidConversion02 -Dtest=!org.apache.hadoop.hive.ql.TestTxnCommands2WithSplitUpdate -Dtest=!org.apache.hadoop.hive.ql.exec.vector.expressions.TestVectorStringExpressions -Dtest=!org.apache.hadoop.hive.ql.exec.TestExecDriver -Dtest=org.apache.hadoop.hive.llap.tezplugins.TestLlapTaskCommunicator --fail-never
                '''
            }
        }
        stage("Publish to Nexus Repository Manager") {
            steps {
                echo "Deploy..."
                withCredentials([usernamePassword(credentialsId: '4b87bd68-ad4c-11ed-afa1-0242ac120002', passwordVariable: 'pass', usernameVariable: 'user')]) {
                    sh 'mvn clean deploy -DskipTests -s settings.xml'
                }
            }        
        }
        stage("Publish tar.gz to Nexus") {
            steps {
                echo "Publish tar.gz..."
                withCredentials([usernamePassword(credentialsId: '4b87bd68-ad4c-11ed-afa1-0242ac120002', passwordVariable: 'pass', usernameVariable: 'user')]) {
                    sh '''
                    curl -v -u $user:$pass --upload-file packaging/target/apache-hive-2.3.10-TDP-0.1.0-SNAPSHOT-bin.tar.gz http://172.19.0.2:8081/repository/maven-tar-files/hive-2.3/apache-hive-2.3.10-TDP-0.1.0-SNAPSHOT-bin-${number}.tar.gz
                    curl -v -u $user:$pass --upload-file packaging/target/apache-hive-2.3.10-TDP-0.1.0-SNAPSHOT-src.tar.gz http://172.19.0.2:8081/repository/maven-tar-files/hive-2.3/apache-hive-2.3.10-TDP-0.1.0-SNAPSHOT-src-${number}.tar.gz
                    '''
                }
            }        
        }
    }
}