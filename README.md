Note:  To Get the coorect font and lines edit and read the Document
############@@@@@@@@@@@@@@$$$$$$$$$$##### Main Installation Steps With QA-Automation ###################$$$$$$$$$$$$@@@@@@@@@@@@@@

Flutter-installations-CI/CD-Pipelines:
Skip to end of metadata
Created by Reddy Prasad, last modified by Reddy Prasad on Sep 10, 2020
Go to start of metadata
Flutter-UFA
Steps:
Step1:  Install Dependencies.
List: install jdk,curl,wget,homebrew,wget,Bash-shell,Unzip, Nodejs,npm.
Link: https://flutter.dev/docs/get-started/install/macos

Step2:
install xcode & Android Studio with required updated Flutter based in iOS or Android Requirements.
Install Flutter&Dart Plugins in Android Studio.
 
Step3: 
Install Flutter & update flutter environment path variables
Android: https://flutter.dev/docs/get-started/flutter-for/android-devs
iOS: https://flutter.dev/docs/get-started/flutter-for/ios-devs
WEB: https://flutter.dev/docs/get-started/codelab

Step4: 
Create and test with any sample flutter App.
 
Step5: 
Implementation with DevOps CI/CD
install DevOps tools:
Git: https://git-scm.com/docs/gittutorial

Fastlane: https://docs.fastlane.tools/

SonarQube: https://docs.sonarqube.org/latest/setup/install-server/
         Download Flutter Sonar Plugin: wget https://github.com/insideapp-oss/sonar-flutter/releases/download/0.3.1/sonar-flutter-plugin-0.3.1.jar

Jfrog: https://www.jfrog.com/confluence/display/JFROG/Installation+and+Configuration

Appcenter: https://docs.microsoft.com/en-gb/appcenter/ 

CI/CD - Pipeline:

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
Git: 
stage('GIT') {
 steps {
 
 git credentialsId: 'b05e6b11-9ff2-4044-b588-c4a9dd337ef8', url: 'https://us.tools.democompany.democompany.com/bitbucket/scm/ufa/reference_architecture.git'
 }
 }
1. Replace the git CredentialsId and Url link.


Flutter-apk-build:
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
1. goto flutter code and build app file.
Flutter-ipa-Build:
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
1. Goto flutter iOS code & build the ipa file by using fast lane tool.
2. Here adhoc_build_ipa lane we are using for the ipa file creation.
Flutter-Web-build:
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
goto flutter code and build web files.
copy latest web files to app.js location and start the web application.
restart the web server and access the weblink in web browser.


Flutter-Test:
stage('flutter-Test'){
            steps {
                sh '''
                pwd
                cd skeleton_app
                flutter test
                '''
            }
        }
1. goto flutter code and run the flutter test to run the test analysis.

Sonar-Analysis:
stage('Sonar-Analyzer'){
      steps {
          sh '''
          cd skeleton_app/build/web
          sonar-scanner \
           -Dsonar.host.url=http://localhost:9000 \
           -Dsonar.login=041a352fdcf1e4efa62737dc9132ebe49b9fb944 \
           -Dsonar.projectKey=Flutter_key \
           -Dsonar.projectKey=Flutter_Key \
           -Dsonar.sources=/Users/democompany/.jenkins/workspace/Flutter_UFA/skeleton_app/build/web \
           -Dsonar.sourceEncoding=UTF-8
          '''
          }
      }



Change the Sonar Project name, Key , Url and Login credentials.
change the sonar sources path.

Artifactory-Upload:
stage('Artifactory_APK_iPA_Upload') {
        steps {
            sh '''
            curl -u flutter:flutter1234 -T /Users/democompany/.jenkins/workspace/REDDY/Flutter-pipeline-test/skeleton_app/build/app/outputs/flutter-apk/app-release.apk  "http://localhost:8081/artifactory/Flutter_Android/"
            curl -u flutter:flutter1234 -T /Users/democompany/Reddy-Data/Fastlane/appcenter/ios/ufa.ipa  "http://localhost:8081/artifactory/Flutter_iOS/"
            curl -u flutter:flutter1234 -T /Users/democompany/Reddy-Data/Fastlane/appcenter/ios/ufa.app.dSYM.zip  "http://localhost:8081/artifactory/Flutter_iOS/"
            '''
            }
        }      

1. Change artifactory UserName , Password and uploading path details.

Appcenter-Upload:
stage('Appcenter-APK&iPA-Upload') {
            steps {
                sh '''
                cd  /Users/democompany/Reddy-Data/Fastlane/appcenter/fastlane
                fastlane appCenterUpload
                fastlane appCenterUpload_ipa
                '''
            }
        }

1. Upload the app & ipa files to appcenter.
2. change the Fastlane lane names.

Automation-QA:
stage('Automation-QA') {
           steps {
               sh '''
               pwd
               cd /Users/democompany/Reddy-Data/Flutter_UFA_Code/reference_architecture/skeleton_app
               dart /Users/democompany/Reddy-Data/Flutter_UFA_Code/reference_architecture/skeleton_app/test_driver/TestMain.dart "tag-@jenkinsdemo | platform-ios,android"
               '''
           }
       }


1. Goto your QA script location and run the required commands to run the QA testing.

Successfull Jenkins Pipeline:


Server Details:
Note: To access the servers please connect to the VPN.
BitBucket: https://democompany.combitbucket/projects/UFA/repos/reference_architecture/browse
Jenkins: http://10.150.22.36:8080/
Jfrog: http://10.150.22.35:8081/artifactory/
Sonar: http://10.150.19.37:9000/projects
AppCenter: https://appcenter.ms/


################################### Second Processs Steps ##################################

Step1:  Install Dependencies.
List: install jdk,curl,wget,homebrew,wget,Bash-shell,Unzip, Nodejs,npm.

Link: https://flutter.dev/docs/get-started/install/macos

Step2:
install xcode & Android Studio with required updated Flutter based in iOS or Android Requirements.
Install Flutter&Dart Plugins in Android Studio.

Step3: 
Install Flutter & update flutter environment path variables
Android: https://flutter.dev/docs/get-started/flutter-for/android-devs
iOS: https://flutter.dev/docs/get-started/flutter-for/ios-devs
WEB: https://flutter.dev/docs/get-started/codelab


Step4: 
Create and test with any sample flutter App.

Step5: 
Implementation with DevOps CI/CD
install DevOps tools Jenkins:
Git: https://git-scm.com/docs/gittutorial

SonarQube: https://docs.sonarqube.org/latest/setup/install-server/
	Download Flutter Sonar Plugin: wget https://github.com/insideapp-oss/sonar-flutter/releases/download/0.3.1/sonar-flutter-plugin-0.3.1.jar


Jfrog: https://www.jfrog.com/confluence/display/JFROG/Installation+and+Configuration

Appcenter: https://docs.microsoft.com/en-gb/appcenter/

Fastlane: https://docs.fastlane.tools/


