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
                        git credentialsId: 'b05e6b11-9ff2-4044-b588-c4a9dd337ef8', url: 'https://us.tools.publicis.sapient.com/bitbucket/scm/ufa/reference_architecture.git'
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
                        cd /Users/publicissapient/Reddy-Data/Fastlane/appcenter
                        rm -rf ios
                        cd /Users/publicissapient/Reddy-Data/Fastlane/appcenter/fastlane
                        fastlane adhoc_build_ipa
                        '''
                    }
                }
        stage('flutter-web-build'){
                    steps {

