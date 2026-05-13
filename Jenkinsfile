pipeline {
    agent any
    tools { maven "M3" }
    stages {
        stage("checkout") {
            steps {
                git branch: "main", url: "git@github.com:phanireddy77/SpringPetClinic.git"
            }
        }
        stage("build") {
            steps {
                sh "mvn package -DskipTests"
            }
        }
        stage("test") {
            steps {
                sh "mvn test"
            }
        }
        stage("deploy") {
            steps {
                 withEnv(['JENKINS_NODE_COOKIE=dontKillMe']) {
                       sh '''
                        #kill any existing job
                        pkill -f "spring-petclinic" || true
                        sleep 3
                        nohup java -jar target/spring-petclinic-*.jar >  /tmp/petclinic.log 2>&1 &
                        echo "App started with PID: $!"
                        sleep 5
                        cat /tmp/petclinic.log
                        '''
                    }
                }
        }
    }
}
