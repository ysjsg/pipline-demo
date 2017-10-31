node('master') {
    def server = Artifactory.server("REPO_56")
    def rtGradle = Artifactory.newGradleBuild()
    def buildInfo
    def buildNumber = VersionNumber([
            versionNumberString: '${BUILD_DATE_FORMATTED, "yyyyMMdd"}.${BUILD_ID}'
    ])

    print "buildNumber:" + buildNumber

    stage('Checkout code') {
        git branch: 'master', credentialsId: '8e5ff08e-5205-4d06-9a26-04a68a81a40e', url: 'https://git.derbysoft.tm/dsone/dsone-pipeline-test.git'
    }

    stage('Artifactory config') {
        rtGradle.tool = "Gradle4" // Tool name from Jenkins configuration
        rtGradle.deployer repo: 'libs-snapshot-local', server: server
        rtGradle.resolver repo: 'libs-snapshot', server: server
        rtGradle.deployer.deployArtifacts = false // Disable artifacts deployment during Gradle run
        buildInfo = Artifactory.newBuildInfo()
    }

    stage('Build') {
        rtGradle.run rootDir: './', buildFile: 'build.gradle', tasks: 'clean build -Pbuild_number=' + buildNumber
    }

    stage('Deploy') {
        rtGradle.run rootDir: './', buildFile: 'build.gradle', tasks: 'artifactoryPublish -Pbuild_number=' + buildNumber, buildInfo: buildInfo
        rtGradle.deployer.deployArtifacts buildInfo
    }

    stage('Publish build info') {
        server.publishBuildInfo buildInfo
    }

    stage('Ansible') {
        ansiblePlaybook playbook: 'ansible.yml'
    }
}