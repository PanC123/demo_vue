pipeline {
    agent any 
    environment {
        DOCKER_REGISTRY_CRED = credentials('dockerRegistry')
        server = Artifactory.server 'art1'
        rtDocker = Artifactory.docker server: server
        buildInfo = Artifactory.newBuildInfo()
        ARTIFACTORY_DOCKER_REGISTRY='pcblog.cn:80/guide-docker-dev-local'
    }
    stages{
        stage ('Clone') {
            step{
                git url: 'https://github.com/PanC123/demo_vue.git'
            }
        }

        stage ('build') {
            steps{
                sh """
                npm config set registry https://registry.npm.taobao.org
                npm install
                npm run build 
                cd dist && zip -r demo_vue.zip *
                """
            }
        }
        stage ('archiveArtifacts') {
            steps{
                archiveArtifacts artifacts: 'dist/demo_vue.zip', followSymlinks: false
            }
        }
        stage ('Build docker image') {
            steps{
                sh "docker login -u $DOCKER_REGISTRY_CRED_USR -p $DOCKER_REGISTRY_CRED_PSW pcblog.cn:80"
                docker.build(ARTIFACTORY_DOCKER_REGISTRY + "/nginx:${env.BUILD_ID}", ".")
            }
        }
        stage ('Push image to Artifactory') {
            steps{
                buildInfo = rtDocker.push ARTIFACTORY_DOCKER_REGISTRY + '/nginx:${env.BUILD_ID}', 'guide-docker-dev-local'
            }
            
        }
        stage ('Publish build info') {
            steps{
                server.publishBuildInfo buildInfo
            }  
        }
    }
}