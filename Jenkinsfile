#!groovy

//def nodes = ['ubuntu14', 'ubuntu16', 'centos6', 'centos7']
def nodes = ['ubuntu14']
def builds = [:]

for (x in nodes) {
    def mynode = x

    // Create a map to pass in to the 'parallel' step so we can fire all the builds at once
    builds[mynode] = {
        node(mynode) {
            stage('Prepare') {
                timeout(time: 10, unit: 'MINUTES') {
                    withEnv(["PATH=/home/jenkins/.linuxbrew/bin:/usr/bin:/bin:/usr/sbin:/sbin", 'HOMEBREW_DEVELOPER=1']) {
                        sh "brew tap kaust-rc/apps"
                    }
                }
                sh "chmod 600 /home/jenkins/.linuxbrew/Library/Taps/kaust-rc/homebrew-apps/*.rb"
            }

            stage('Test') {
                timeout(time: 1, unit: 'HOURS') {
                    withEnv(["PATH=/home/jenkins/.linuxbrew/bin:/usr/bin:/bin:/usr/sbin:/sbin", 'HOMEBREW_DEVELOPER=1']) {
                        sh "brew test-bot --tap=kaust-rc/apps --junit weather"
                    }
                    junit 'brew-test-bot.xml'
                }
            }

            post {
                success {
                    slackSend channel: '#devops', color: 'good', message: '${env.JOB_NAME} (${env.BUILD_NUMBER}) was successfully built. Link to build: ${env.BUILD_URL}.'
                }

                failure {
                    slackSend channel: '#devops', color: 'bad', message: '${env.JOB_NAME} (${env.BUILD_NUMBER}) failed; please look into it now! Link to build: ${env.BUILD_URL}.'
                }

                unstable {
                    slackSend channel: '#devops', color: 'warning', message: '${env.JOB_NAME} (${env.BUILD_NUMBER}) is unstable; someone should check ASAP. Link to build: ${env.BUILD_URL}.'
                }
            }
        }
    }
}

parallel builds
