#!groovy

def warn = { msg ->
  if (!currentBuild.description) {
    currentBuild.description += ''
  }
  else if (currentBuild.description.substring(currentBuild.description.length() - 1) != '\n') {
    currentBuild.description += '<br />\n'
  }
  currentBuild.description += "warning: ${msg}<br />\n"
}

ansiColor('xterm') {
  timestamps {
    timeout(90) {
      node('NODE_JS_BUILDER') {

        def packageJsonVersion
        def skipAction
        def skipTests = false

        try {
          // We're not currently using structured output, just relying
          // on exict codes, so the build is a success until something
          // throws
          currentBuild.result = 'SUCCESS'

          // Set the description to blank so we can use +=
          currentBuild.description = ''

          stage('checkout') {
            checkout scm

            sh 'git config user.email spark-js-sdk.gen@cisco.com'
            sh 'git config user.name Jenkins'

            try {
              pusher = sh script: 'git show --quiet --format=%ae HEAD', returnStdout: true
              currentBuild.description += "Validating push from ${pusher}"
            }
            catch (err) {
              currentBuild.description += 'Could not determine pusher';
            }

            sshagent(['d8533977-c4c5-4e2b-938d-ae7fcbe27aac']) {
              // return the exit code because we don't care about failures
              sh script: 'git remote add upstream git@github.com:webex/web-styleguide.git', returnStatus: true
              // Make sure local tags don't include failed releases
              sh 'git tag -l | xargs git tag -d'
              sh 'git gc'
              sh 'git fetch upstream --tags'
            }

            changedFiles = sh script: 'git diff --name-only upstream/master..$(git merge-base HEAD upstream/master)', returnStdout: true
            if (changedFiles.contains('Jenkinsfile')) {
              currentBuild.description += "Jenkinsfile has been updated in master. Please rebase and push again."
              error(currentBuild.description)
            }

            sh 'git checkout upstream/master'
            try {
              sh "git merge --ff ${GIT_COMMIT}"
            }
            catch (err) {
              currentBuild.description = 'not possible to fast forward'
              throw err;
            }
          }

          stage('Clean') {
            sh 'rm -rf node_modules'
          }

          stage('Install') {
            withCredentials([
              string(credentialsId: 'JS_SDK_NPM_PUBLISH_TOKEN', variable: 'JS_SDK_NPM_PUBLISH_TOKEN')
            ]) {
              sh 'echo \'//registry.npmjs.org/:_authToken=${JS_SDK_NPM_PUBLISH_TOKEN}\' >> .npmrc'
              sh '''#!/bin/bash -e
              source ~/.nvm/nvm.sh
              nvm install v8.11.3
              nvm use v8.11.3
              npm install -g npm@6.4.1
              npm install
              git checkout .npmrc
              '''
            }
          }

          stage('Static Analysis') {
            withCredentials([
              string(credentialsId: 'JS_SDK_NPM_PUBLISH_TOKEN', variable: 'JS_SDK_NPM_PUBLISH_TOKEN')
            ]) {
              sh '''#!/bin/bash -e
              source ~/.nvm/nvm.sh
              nvm use v8.11.3
              npm run static-analysis
              '''
            }
          }

          stage('Bump version') {
            withCredentials([
              string(credentialsId: 'JS_SDK_NPM_PUBLISH_TOKEN', variable: 'JS_SDK_NPM_PUBLISH_TOKEN')
            ]) {
              sh '''#!/bin/bash -e
              source ~/.nvm/nvm.sh
              nvm use v8.11.3
              npm run release -- --release-as patch --no-verify
              version=`grep "version" package.json | head -1 | awk -F: '{ print $2 }' | sed 's/[", ]//g'`
              echo $version > .version
              '''
              packageJsonVersion = readFile '.version'
            }
          }

          stage('Check for No Push') {
            try {
              noPushCount = sh script: 'git log upstream/master.. | grep -c "#no-push"', returnStdout: true
              if (noPushCount != '0') {
                currentBuild.result = 'ABORTED'
                currentBuild.description += 'Aborted: git history includes #no-push'
              }
            }
            catch (err) {
              // ignore. turns out that when there are zero #no-push
              // commits, sh throws. This should be improved at some point,
              // but gets the job done for now
            }
          }

          if (currentBuild.result == 'SUCCESS'){

            stage('Push to github'){
              sshagent(['d8533977-c4c5-4e2b-938d-ae7fcbe27aac']) {
                sh "git push upstream HEAD:master && git push --tags upstream"
              }
            }

            stage('Publish to NPM') {
              withCredentials([
                string(credentialsId: 'JS_SDK_NPM_PUBLISH_TOKEN', variable: 'JS_SDK_NPM_PUBLISH_TOKEN')
              ]) {
                try {
                  sh 'echo \'//registry.npmjs.org/:_authToken=${WEBSTYLES_NPM_TOKEN}\' >> $HOME/.npmrc'
                  // Publish
                  echo ''
                  echo 'Reminder: E403 errors below are normal. They occur for any package that has no updates to publish'
                  echo ''
                  sh '''#!/bin/bash -e
                  source ~/.nvm/nvm.sh
                  nvm use v8.11.3
                  npm run publish:components
                  rm $HOME/.npmrc
                  '''
                }
                catch (error) {
                  warn("failed to publish to npm ${error.toString()}")
                }
              }
            }
          }
          cleanup()
        }
        catch (error) {
         // Sometimes an exception can get thrown without changing the build result
         // from success. If we reach this point and the result is not UNSTABLE, then
         // we need to make sure it's FAILURE
          if (currentBuild.result != 'UNSTABLE') {
            currentBuild.result = 'FAILURE'
          }
          cleanup()
          throw error
        }
      }
    }
  }
}
