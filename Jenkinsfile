def gitCommit() {
    sh "git rev-parse HEAD > GIT_COMMIT"
    def gitCommit = readFile('GIT_COMMIT').trim()
    sh "rm -f GIT_COMMIT"
    return gitCommit
}

node {
    // Checkout source code from Git
    stage 'Checkout'
    checkout scm

    // Build Docker image
    stage 'Build'
    sh "docker build -t vnyuser/vny:${gitCommit()} ."

    // Log in and push image to GitLab
    stage 'Publish'
    withCredentials(
        [[
            $class: 'UsernamePasswordMultiBinding',
            credentialsId: 'vnyuser',
            passwordVariable: 'DOCKERHUB_PASSWORD',
            usernameVariable: 'DOCKERHUB_USERNAME'
        ]]
    ) {
        sh "docker login -u '${env.DOCKERHUB_USERNAME}' -p '${env.DOCKERHUB_PASSWORD}' -e demo@mesosphere.com"
        sh "docker push vnyuser/vny:${gitCommit()}"
    }

    stage 'Deploy'

    marathon(
        url: 'http://marathon.mesos:8080',
        forceUpdate: true,
        credentialsId: 'dcos-token',
        filename: 'marathon.json',
        appId: 'vnyuser',
        docker: "vnyuser/vny:${gitCommit()}".toString()
    )
}
