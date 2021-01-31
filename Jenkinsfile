pipeline {
    agent none
    stages {
        stage('Compile and Test') {
            agent any
            steps {
                sh "mvn --batch-mode package" 
            }
        }

        stage('Publish Tests Results') {
            agent any
            steps {
               echo 'Archive Unit Test Results'
               step([$class: 'JUnitResultArchiver', testResults: 'target/surefire-reports/TEST-*.xml'])
            }
        }
        
        stage('Create and Publish Docker Image'){
            agent any
            steps{
                script {
                    env.GITHUB_USER = sh(script: "sed -n '1p' /tmp/shortname.txt",returnStdout: true).trim()
                    env.SHORT_COMMIT= env.GIT_COMMIT[0..7]
                    env.TAG_NAME="docker.pkg.github.com/$GITHUB_USER/pet-clinic/petclinic:$SHORT_COMMIT".toLowerCase()
                }
                sh "docker build -t $TAG_NAME -f Dockerfile.deploy ."
                sh "sed -n '2p' /tmp/shortname.txt | docker login https://docker.pkg.github.com -u $GITHUB_USER --password-stdin"
                sh "docker push $TAG_NAME"
            }
        }

        stage('Deploy Development') {
            agent any
            steps {
                sh "chmod +x deploy.sh"
                sh "./deploy.sh dev $TAG_NAME"
            }
        }
        

    }
}
