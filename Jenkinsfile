podTemplate(containers: [
    containerTemplate(
        name: 'tdp-builder', 
        image: 'yanisbariteau/tdp-builder:jenkins', 
        command: 'sleep', 
        args: '30d'
        )
  ]) {

    node(POD_LABEL) {
        container('tdp-builder') {
            environment {
                number="${currentBuild.number}"
            }
            stage('Git Clone') {
                echo "Cloning..."
                git branch: 'branch-1.2-TDP-alliage-k8s', url: 'https://github.com/Yanis77240/hive'
            }   
            stage ('Build') {
                echo "Building..."
                sh '''
                mvn clean install -Pdist -Phadoop-2 -DskipTests
                '''
            }
            stage('Test') {
                echo "Testing..."
                withEnv(["number=${currentBuild.number}"]) {
                    withCredentials([usernamePassword(credentialsId: '4b87bd68-ad4c-11ed-afa1-0242ac120002', passwordVariable: 'pass', usernameVariable: 'user')]) {
                        sh 'mvn clean test -Phadoop-2 --fail-never'
                        sh 'mvn surefire-report:report-only  -Daggregate=true'
                        sh 'curl -v -u $user:$pass --upload-file target/site/surefire-report.html http://10.110.4.212:8081/repository/test-reports/hive-1.2/surefire-report-${number}.html'
                    }
                }
            }
            stage('Deliver') {
                echo "Deploy..."
                withCredentials([usernamePassword(credentialsId: '4b87bd68-ad4c-11ed-afa1-0242ac120002', passwordVariable: 'pass', usernameVariable: 'user')]) {
                    sh 'mvn clean deploy -Pdist -Phadoop-2 -DskipTests -s settings.xml'
                }
            }
            stage("Publish tar.gz to Nexus") {
                echo "Publish tar.gz..."
                withEnv(["number=${currentBuild.number}"]) {
                    withCredentials([usernamePassword(credentialsId: '4b87bd68-ad4c-11ed-afa1-0242ac120002', passwordVariable: 'pass', usernameVariable: 'user')]) {
                        sh '''
                        curl -v -u $user:$pass --upload-file packaging/target/apache-hive-1.2.3-TDP-0.1.0-SNAPSHOT-bin.tar.gz http://10.110.4.212:8081/repository/maven-tar-files/hive-1.2/apache-hive-1.2.3-TDP-0.1.0-SNAPSHOT-bin-${number}.tar.gz
                        curl -v -u $user:$pass --upload-file packaging/target/apache-hive-1.2.3-TDP-0.1.0-SNAPSHOT-src.tar.gz http://10.110.4.212:8081/repository/maven-tar-files/hive-1.2/apache-hive-1.2.3-TDP-0.1.0-SNAPSHOT-src-${number}.tar.gz
                        '''
                    }
                }
            }       
        }
    }
}