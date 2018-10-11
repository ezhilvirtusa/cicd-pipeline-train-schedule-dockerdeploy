pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
   
 stage("Build Docker Image"){
when{
branch "master"
}
steps{
script{
app=docker.build("ezhilc/trainapp")
app.inside{
sh 'echo $(curl localhost:8080)'
}
}
}
}

stage("Push Docker Image"){
when{
branch "master"
}
steps{
script{
    docker.withRegistry('https://registry.hub.docker.com','DockerID'){
app.push("${env.BUILD_NUMBER}")
app.push("latest")
}
}
}
}

stage("Deploy to prod"){
when{
branch "master"
}
steps {
input 'Do you want to deploy to prod?'
milestone(1)
withCredentials([usernamePassword(credentialsId: 'webserver_login',usernameVariable:'USERNAME',passwordVariable: 'USERPASS')]) {
script {
sh "sshpass -p 'USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod \"docker pull ezhilc/trainapp:${env.BUILDNUMBER}\""
try{
sh  "sshpass -p 'USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod \"docker stop trainapp\""
sh  "sshpass -p 'USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod \"docker rm trainapp\""
}
catch (eer){
echo : 'caught error:$err'
}
sh  "sshpass -p 'USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod \"docker run --restart always --name trainapp -p 8080:8080 -d ezhilc/trainapp:${env.BUILDNUMBER}\""
}
}
}
}
}
}

