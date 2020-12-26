Jenkins-Pipeline-Code:

pipeline {
    agent {
        node {
            label 'master'
        }
    }
    //properties([gitLabConnection(''), pipelineTriggers([pollSCM('H/5 * * * *')])])
    stages{
        stage('GIT') {
            steps {
            git credentialsId: 'b05e6b11-9ff2-4044-b588-c4a9dd337ef8', url: 'https://us.tools.democompany.democompany.com/bitbucket/scm/ufa/reference_architecture.git'
            }
        }
        stage('flutter-apk-build'){
            steps {
                sh '''
                pwd
                ls -ll
                cd skeleton_app
                ls -ll
                flutter --version
                flutter build apk --release
                '''
            }
        }
    stage('flutter-ipa-build'){
            steps {
                sh '''
                pwd
                echo "iOS-Build stage Running"
                cd skeleton_app
                flutter pub get
                rm -rf ios/Podfile
                flutter build ios --release
                cd /Users/democompany/Reddy-Data/Fastlane/appcenter
                rm -rf ios
                cd /Users/democompany/Reddy-Data/Fastlane/appcenter/fastlane
                fastlane adhoc_build_ipa
                '''
            }
        }
    stage('flutter-web-build'){
            steps {
                sh '''
                pwd
                cd skeleton_app
                flutter pub get
                rm -rf ios/Podfile
                flutter config --enable-web
                flutter build web --release
                cp -R /Users/democompany/.jenkins/workspace/Flutter_UFA/skeleton_app/web/ /Users/democompany/Workspace/NodeJS/skeleton_app
                cd /Users/democompany/Workspace/NodeJS/skeleton_app
                pwd
                ls -ll
               nodejs_pid=$(ps a | grep -v grep | grep app.js | cut -d " " -f1)
               echo "$nodejs_pid"
               # Stopping node.js service
               if [ -z "$nodejs_pid" ]
                then
                    echo "$nodejs_pid is empty"
                else
                    Kill -9 "$nodejs_pid"
                fi
               # Starting node.js service
               cd /Users/democompany/Workspace/NodeJS/skeleton_app
               npm install
               nohup node app.js > /dev/null &
               exit
               # node.js PID after starting
               nodejs_pid=$(ps a | grep -v grep | grep app.js | cut -d " " -f1)
               echo "$nodejs_pid"
                '''
            }
        }
    stage('flutter-Test'){
            steps {
                sh '''
                pwd
                cd skeleton_app
                flutter test
                '''
            }
        }
    stage('Sonar-Analyzer'){
        steps {
            sh '''
            cd skeleton_app/build/web
            sonar-scanner \
             -Dsonar.host.url=http://localhost:9000 \
             -Dsonar.login=041a352fdcf1e4efa62737dc9132ebfb944 \
             -Dsonar.projectKey=Flutter_key \
             -Dsonar.projectKey=Flutter_Key \
             -Dsonar.sources=/Users/democompany/.jenkins/workspace/Flutter_UFA/skeleton_app/build/web \
             -Dsonar.sourceEncoding=UTF-8
            '''
            }
        }
    stage('Artifactory_APK_iPA_Upload') {
        steps {
            sh '''
            curl -u flutter:flutter1234 -T /Users/democompany/.jenkins/workspace/REDDY/Flutter-pipeline-test/skeleton_app/build/app/outputs/flutter-apk/app-release.apk  "http://localhost:8081/artifactory/Flutter_Android/"
            curl -u flutter:flutter1234 -T /Users/democompany/Reddy-Data/Fastlane/appcenter/ios/ufa.ipa  "http://localhost:8081/artifactory/Flutter_iOS/"
            curl -u flutter:flutter1234 -T /Users/democompany/Reddy-Data/Fastlane/appcenter/ios/ufa.app.dSYM.zip  "http://localhost:8081/artifactory/Flutter_iOS/"
            '''
            }
        }
    stage('Appcenter-APK&iPA-Upload') {
            steps {
                sh '''
                cd  /Users/democompany/Reddy-Data/Fastlane/appcenter/fastlane
                fastlane appCenterUpload
                fastlane appCenterUpload_ipa
                '''
            }
        }
        stage('Automation-QA') {
            steps {
                sh '''
                pwd
                cd /Users/democompany/Reddy-Data/Flutter_UFA_Code/reference_architecture/skeleton_app
                dart /Users/democompany/Reddy-Data/Flutter_UFA_Code/reference_architecture/skeleton_app/test_driver/TestMain.dart "tag-@jenkinsdemo | platform-ios,android"
                '''
            }
        }
    }
}
