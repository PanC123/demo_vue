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
        """
    }

    stage ('Docker login') {
        sh 'docker login -u jfrog -p JFrog@123 pcblog.cn:80'
    }

    stage ('Build docker image') {
        docker.build(ARTIFACTORY_DOCKER_REGISTRY + '/tomcat:jdk8-openjdk-slim', '.')
    }

    stage ('Push image to Artifactory') {
        buildInfo = rtDocker.push ARTIFACTORY_DOCKER_REGISTRY + '/tomcat:jdk8-openjdk-slim', 'guide-docker-dev-local'
    }

    stage ('Publish build info') {
        server.publishBuildInfo buildInfo
    }
}