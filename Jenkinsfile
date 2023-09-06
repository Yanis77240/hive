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
            stage('Git Clone') {
                echo "Cloning..."
                git branch: 'branch-2.3-TDP-alliage-k8s', url: 'https://github.com/Yanis77240/hive'
            }
            stage('Chose comparison') {
                withEnv(["file=${input message: 'Select file in http://10.10.10.11:30000/repository/java-test-reports/', parameters: [string('number of results file')]}"]) {
                    withEnv(["number=${currentBuild.number}"]) {
                        sh '''
                        cd test-comparison
                        curl -v http://repository/component-test-comparison/hive2/${file} > ${file}
                        python3 src/python/comparison_file_check.py ${file}
                        echo "python3 src/python/main.py 3.0.0 ${number} ${file}" > transformation.sh
                        chmod 777 transformation.sh
                        '''
                    }
                }
            }
            stage ('Build') {
                echo "Building..."
                sh '''
                mvn clean install -Pdist -DskipTests
                '''
            }
            stage('Test') {
                echo "Testing..."
                withCredentials([usernamePassword(credentialsId: '4b87bd68-ad4c-11ed-afa1-0242ac120002', passwordVariable: 'pass', usernameVariable: 'user')]) {
                    withEnv(["number=${currentBuild.number}"]) {
                        /* Perform the tests and the surefire reporting*/
                        sh '''
                        mvn clean test -Dstyle.color=never --fail-never clean test | tee output.txt
                        '''
                        sh 'mvn surefire-report:report-only  -Daggregate=true'
                        sh 'curl -v -u $user:$pass --upload-file target/site/surefire-report.html http://10.110.4.212:8081/repository/test-reports/hive-2.3/surefire-report-${number}.html'
                        /* extract the scalatest-plugin data and java-test data output and remove all color signs */
                        sh'./test-comparison/src/grep_commands/grep-surefire-3.0.0.sh'
                        /* Perform the data transformation and the comparison*/
                        sh '''
                        cd test-comparison
                        ./transformation.sh
                        ./src/decision.sh ${number}
                        curl -v -u $user:$pass --upload-file results-${number}.json http://10.110.4.212:8081/repository/component-test-comparison/hive2/results-${number}.json
                        '''
                    }
                }
            }
            stage('Deliver') {
                echo "Deploy..."
                withCredentials([usernamePassword(credentialsId: '4b87bd68-ad4c-11ed-afa1-0242ac120002', passwordVariable: 'pass', usernameVariable: 'user')]) {
                    sh 'mvn clean deploy -Pdist -DskipTests -s settings.xml'
                }
            }
            stage("Publish tar.gz to Nexus") {
                echo "Publish tar.gz..."
                withEnv(["number=${currentBuild.number}"]) {
                    withCredentials([usernamePassword(credentialsId: '4b87bd68-ad4c-11ed-afa1-0242ac120002', passwordVariable: 'pass', usernameVariable: 'user')]) {
                        sh '''
                        curl -v -u $user:$pass --upload-file packaging/target/apache-hive-2.3.10-TDP-0.1.0-SNAPSHOT-bin.tar.gz http://10.110.4.212:8081/repository/maven-tar-files/hive-2.3/apache-hive-2.3.10-TDP-0.1.0-SNAPSHOT-bin-${number}.tar.gz
                        curl -v -u $user:$pass --upload-file packaging/target/apache-hive-2.3.10-TDP-0.1.0-SNAPSHOT-src.tar.gz http://10.110.4.212:8081/repository/maven-tar-files/hive-2.3/apache-hive-2.3.10-TDP-0.1.0-SNAPSHOT-src-${number}.tar.gz
                        '''
                    }
                }
            }       
        }
    }
}  