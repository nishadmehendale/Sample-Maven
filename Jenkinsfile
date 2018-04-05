
pipeline {
    agent any
    tools {
        jdk 'jdk8'
        maven 'maven3'
    }
    
    stages {


        stage('download_stash_code') {
            steps {
                checkout scm
            }
        }

        stage('clean') {
            steps {
                sh "mvn clean"
            }
        }

        // stage('munit') {
        //     steps {
        //        sh 'mvn -U clean test cobertura:cobertura -Dcobertura.report.format=xml'
        //         junit '**/target/*-reports/TEST-*.xml'
        //         step([$class: 'CoberturaPublisher', coberturaReportFile: 'target/site/cobertura/coverage.xml'])
        //     }
        //  }


        // stage('Sonar') {
        //     steps {
        //         sh "mvn sonar:sonar -Dsonar.host.url=${env.SONARQUBE_HOST}"
        //     }
        // }

        // stage('publish munit result') {
        //     steps {
        //         publish_html()
        //     }
        // }

        stage('push to artifactory') {

            steps {

            script{




                    fetch_tags()



               if (env.BRANCH_NAME == 'master') {
                    echo 'I only execute on the master branch'

                    configFileProvider([configFile(fileId: 'our_settings', variable: 'SETTINGS')]) {
                    sh "mvn -s $SETTINGS deploy -DskipTests -Dbuild.version=${gitTagLatest()}.${env.BUILD_NUMBER} -Dartifactory_url=${env.ARTIFACTORY_URL} -Dartifactory_name=${env.ARTIFACTORY_NAME}"
                    }

                } else if (env.BRANCH_NAME == 'develop') {
                    echo 'I only execute on the develop branch'


                    configFileProvider([configFile(fileId: 'our_settings', variable: 'SETTINGS')]) {
                    sh "mvn -s $SETTINGS deploy -DskipTests -Dbuild.version=${gitTagLatest()}.${env.BUILD_NUMBER} -Dartifactory_url=${env.ARTIFACTORY_URL} -Dartifactory_name=${env.ARTIFACTORY_NAME}"
                    }
                }
                else {
                    echo 'I execute elsewhere'
                }

            }

            }
        }


        // stage('tag the build') {
        //     steps { 


        //             withCredentials([
        //                 [$class: 'UsernamePasswordMultiBinding', credentialsId: '83aa9347-b473-4a44-8397-8a3822630839', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASS'],
        //                     ]){     
        //                             sh """(
        //                             git remote set-url origin https://${GIT_USER}:${GIT_PASS}@bitbucket.org/cfsintnadev/app-dev-flights-ubuntu-ws.git
        //                             git config --global user.email 'vrnvikas1994@gmail.com'
        //                             git config --global user.name ${GIT_USER}
        //                             git tag -a ${gitTagLatest()}.${env.BUILD_NUMBER} -m 'build-${env.BUILD_NUMBER}'
        //                             git push --force origin refs/tags/${gitTagLatest()}.${env.BUILD_NUMBER}:refs/tags/${gitTagLatest()}.${env.BUILD_NUMBER}
        //                             )"""
        //                         }
                                

        //     }
        // }


       //   stage('deploy app'){
       //      steps { 
       //          script {
  
       //              sshagent (credentials: ['712e5b00-8e63-4237-9065-c69ef3e4cae9']) {
       //              sh "ssh -o StrictHostKeyChecking=no -l ec2-user ${env.MULE_SERVER} ./download_artifact.sh ${gitTagLatest()}.${env.BUILD_NUMBER}-SNAPSHOT ${env.ARTIFACTORY_URL}"
       //              logstashSend failBuild: true
       //              }
       //          }
       //      }
       // }        
        
    }
    
}

/** @return The tag name, or `null` if the current commit isn't a tag. */
String gitTagName() {
    commit = getCommit()
    if (commit) {
        desc = sh(script: "git describe --tags ${commit}", returnStdout: true)?.trim()
            return desc
    }
    return null
}

/** @return The tag version */
String gitTagLatest() {

    sha = sh(script: "git rev-list --tags --max-count=1", returnStdout: true)?.trim()
    longTag = sh(script: "git describe --tags ${sha}", returnStdout: true)?.trim()

    shortTag = longTag.substring(0,longTag.lastIndexOf("."))
    return shortTag
}
 
/** @return The tag message, or `null` if the current commit isn't a tag. */
String gitTagMessage() {
    name = gitTagName()
    msg = sh(script: "git tag -n10000 -l ${name}", returnStdout: true)?.trim()
    if (msg) {
        return msg.substring(name.size()+1, msg.size())
    }
    return null
}
 
String getCommit() {
    return sh(script: 'git rev-parse HEAD', returnStdout: true)?.trim()
}
 
@NonCPS
boolean isTag(String desc) {
    match = desc =~ /.+-[0-9]+-g[0-9A-Fa-f]{6,}$/
    result = !match
    match = null // prevent serialisation
    return result
}

def publish_html()
{   
    def exists = fileExists 'target/munit-reports/coverage/summary.html'
    if (exists) 
    {
        echo 'html report file is generated'
        publishHTML (target: [
        allowMissing: false,
        alwaysLinkToLastBuild: false,
        keepAll: true,
        reportDir: 'target/munit-reports/coverage',
        reportFiles: 'summary.html',
        reportName: "Coverage Report" ])
    } 
        
}


def fetch_tags(){

                                        withCredentials([
                        [$class: 'UsernamePasswordMultiBinding', credentialsId: '83aa9347-b473-4a44-8397-8a3822630839', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASS'],
                            ]){     
                                    sh """(
                                    git remote set-url origin https://${GIT_USER}:${GIT_PASS}@bitbucket.org/cfsintnadev/app-dev-flights-ubuntu-ws.git
                                    git config --global user.email 'vrnvikas1994@gmail.com'
                                    git config --global user.name ${GIT_USER}
                                    git fetch --tags --progress origin +refs/heads/*:refs/remotes/origin/* --prune
                                    )"""
                                }

}