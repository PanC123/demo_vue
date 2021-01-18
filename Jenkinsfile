node {
    def server = Artifactory.server 'art1'
    def rtDocker = Artifactory.docker server: server
    def buildInfo = Artifactory.newBuildInfo()
    def ARTIFACTORY_DOCKER_REGISTRY='pcblog.cn:80/guide-docker-dev-local'
    environment {
        DOCKER_REGISTRY_CRED = credentials('dockerRegistry')
    }
    stage ('Clone') {
        git url: 'https://github.com/PanC123/demo_vue.git'
    }

    stage ('build') {
        sh """
        npm config set registry https://registry.npm.taobao.org
        npm install
        npm run build 
        cd dist && zip -r demo_vue.zip *
        """
    }
    stage ('archiveArtifacts') {
        archiveArtifacts artifacts: 'dist/demo_vue.zip', followSymlinks: false
    }

    stage ('Docker login') {
        sh "docker login -u $DOCKER_REGISTRY_CRED_USR -p $DOCKER_REGISTRY_CRED_PSW pcblog.cn:80"
    }

    stage ('Build docker image') {
        docker.build(ARTIFACTORY_DOCKER_REGISTRY + "/nginx:${env.BUILD_ID}", ".")
    }

    stage ('Push image to Artifactory') {
        buildInfo = rtDocker.push ARTIFACTORY_DOCKER_REGISTRY + '/nginx:${env.BUILD_ID}', 'guide-docker-dev-local'
    }

    stage ('Publish build info') {
        server.publishBuildInfo buildInfo
    }
}