node {
    def server = Artifactory.server 'art1'
    def rtDocker = Artifactory.docker server: server
    def buildInfo = Artifactory.newBuildInfo()
    def ARTIFACTORY_DOCKER_REGISTRY='pcblog.cn:80/guide-docker-dev-local'
    stage ('Clone') {
        git url: 'https://github.com/PanC123/demo_vue.git'
    }

    stage ('build') {
        sh """
        npm config set registry https://registry.npm.taobao.org
        npm install
        npm run build 
        cd dist && zip -r demo_vue.zip *
        mv demo_vue.zip ../
        """
    }
    stage ('archiveArtifacts') {
        archiveArtifacts artifacts: 'demo_vue.zip', followSymlinks: false
    }

    stage ('Docker Image Build') {
        docker.withRegistry('http://pcblog.cn:80','dockerRegistry') {
            docker.build(ARTIFACTORY_DOCKER_REGISTRY + "/nginx:${env.BUILD_ID}", ".")
        }
    }

    stage ('Push image to Artifactory') {
        buildInfo = rtDocker.push ARTIFACTORY_DOCKER_REGISTRY + "/nginx:${env.BUILD_ID}", "guide-docker-dev-local"
        server.publishBuildInfo buildInfo
    }

    stage ('Deploy Image') {
        sh """
        export TAG=${env.BUILD_ID}
        docker-compose stop && docker-compose up -d
        """
    }
}