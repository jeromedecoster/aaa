

pipeline {
   agent any

   environment {
        GITHUB = credentials('github')
        GIT_URL = 'https://github.com/jeromedecoster/aaa.git'
   }
   
   stages {
      stage('Clone') {
         steps {
            cleanWs()
            git 'https://github.com/jeromedecoster/aaa.git'
            slackSend color: 'good', message: 'Starting *API Testing* Job'
            sh '''
            # git clone ${GIT_URL} project
            cd aaa
            git branch develop
            '''
         }
      }
      stage('Test') {
         steps {
            sh '''#!/bin/bash
            cd project
            if [[ $(bash a.sh) = 'ok' ]]
            then
                echo OUI
            else
                echo NON
                exit 1
            fi
            '''
         }
         post {
            success {
                slackSend color: 'good', message: "${currentBuild.fullDisplayName}: stage(Test) *success*"
            }
            failure { 
                slackSend color: 'danger', message: "${currentBuild.fullDisplayName}: stage(Test) *failure*"
            }
         }
      }
      stage('Release') {
         steps {
            withCredentials([string(credentialsId: 'github', variable: 'TOKEN')]) {
            sh '''#!/bin/bash
            cd project
            LAST_LOG=$(git log --format='%H' --max-count=1 master)
            LAST_MERGE=$(git log --format='%H' --merges --max-count=1 master)
            LAST_MSG=$(git log --format='%s' --max-count=1 master)
            VERSION=$(echo $LAST_MSG | grep --only-matching 'v\\?[0-9]\\+\\.[0-9]\\+\\(\\.[0-9]\\+\\)\\?')
            echo "VERSION:$VERSION"
            if [[ $LAST_LOG == $LAST_MERGE && -n $VERSION ]]
            then
                DATA='{
                    "tag_name": "'$VERSION'",
                    "target_commitish": "master",
                    "name": "'$VERSION'",
                    "body": "Test description",
                    "draft": false,
                    "prerelease": false
                }'

                curl --data "$DATA" "https://api.github.com/repos/jeromedecoster/aaa/releases?access_token=$TOKEN"
            fi
            '''
            }
         }
      }
   }
}
