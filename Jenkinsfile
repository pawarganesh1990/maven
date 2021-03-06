/*
 * Licensed to the Apache Software Foundation (ASF) under one
 * or more contributor license agreements.  See the NOTICE file
 * distributed with this work for additional information
 * regarding copyright ownership.  The ASF licenses this file
 * to you under the Apache License, Version 2.0 (the
 * "License"); you may not use this file except in compliance
 * with the License.  You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing,
 * software distributed under the License is distributed on an
 * "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
 * KIND, either express or implied.  See the License for the
 * specific language governing permissions and limitations
 * under the License.
 */

properties([buildDiscarder(logRotator(artifactNumToKeepStr: '5', numToKeepStr: env.BRANCH_NAME=='master'?'10':'5'))])

def itBranch='master'

try {
node('ubuntu') {
    stage 'Checkout'
    def MAVEN_BUILD=tool name: 'Maven 3.3.9', type: 'hudson.tasks.Maven$MavenInstallation'
    echo "Driving build and unit tests using Maven $MAVEN_BUILD"
    def JAVA7_HOME=tool name: 'JDK 1.7 (latest)', type: 'hudson.model.JDK'
    echo "Running build and unit tests with Java $JAVA7_HOME"
    dir('build') {
        checkout scm
        def WORK_DIR=pwd()
        stage 'Build / Unit Test'
        withEnv(["PATH+MAVEN=$MAVEN_BUILD/bin","PATH+JDK=$JAVA7_HOME/bin"]) {
            sh "mvn clean verify -B -U -e -fae -V -Dmaven.test.failure.ignore=true -Dmaven.repo.local=$WORK_DIR/.repository"
        }
        dir ('apache-maven/target') {
            sh "mv apache-maven-*-bin.zip apache-maven-dist.zip"
            stash includes: 'apache-maven-dist.zip', name: 'dist'
        }
        junit allowEmptyResults: true, testResults:'**/target/*-reports/*.xml'
    }
}

stage 'Integration Test'
parallel linuxJava7:{
        node('ubuntu') {
            def MAVEN_NIX_J7=tool name: 'Maven 3.3.9', type: 'hudson.tasks.Maven$MavenInstallation'
            echo "Driving integration tests using Maven $MAVEN_NIX_J7"
            def JAVA_NIX_J7=tool name: 'JDK 1.7 (latest)', type: 'hudson.model.JDK'
            echo "Running integration tests with Java $JAVA_NIX_J7"
            dir('test') {
                def WORK_DIR=pwd()
                git(url:'https://git-wip-us.apache.org/repos/asf/maven-integration-testing.git', branch: itBranch)
                sh "rm -rvf $WORK_DIR/apache-maven-dist.zip $WORK_DIR/it-local-repo"
                unstash 'dist'
                withEnv(["PATH+MAVEN=$MAVEN_NIX_J7/bin","PATH+JDK=$JAVA_NIX_J7/bin"]) {
                    sh "mvn clean install -Prun-its -B -U -V -Dmaven.test.failure.ignore=true -Dmaven.repo.local=$WORK_DIR/it-local-repo -DmavenDistro=$WORK_DIR/apache-maven-dist.zip"
                }
                junit allowEmptyResults: true, testResults:'core-it-support/**/target/*-reports/*.xml,core-it-suite/target/*-reports/*.xml'
            }
        }
    },linuxJava8: {
        node('ubuntu') {
            def MAVEN_NIX_J8=tool name: 'Maven 3.3.9', type: 'hudson.tasks.Maven$MavenInstallation'
            echo "Driving integration tests using Maven $MAVEN_NIX_J8"
            def JAVA_NIX_J8=tool name: 'JDK 1.8 (latest)', type: 'hudson.model.JDK'
            echo "Running integration tests with Java $JAVA_NIX_J8"
            dir('test') {
                def WORK_DIR=pwd()
                git(url:'https://git-wip-us.apache.org/repos/asf/maven-integration-testing.git', branch: itBranch)
                sh "rm -rvf $WORK_DIR/apache-maven-dist.zip $WORK_DIR/it-local-repo"
                unstash 'dist'
                withEnv(["PATH+MAVEN=$MAVEN_NIX_J8/bin","PATH+JDK=$JAVA_NIX_J8/bin"]) {
                    sh "mvn clean install -Prun-its -B -U -V -Dmaven.test.failure.ignore=true -Dmaven.repo.local=$WORK_DIR/it-local-repo -DmavenDistro=$WORK_DIR/apache-maven-dist.zip"
                }
                junit allowEmptyResults: true, testResults:'core-it-support/**/target/*-reports/*.xml,core-it-suite/target/*-reports/*.xml'
            }
        }
    }, winJava7: {
        node('Windows') {
            def MAVEN_WIN_J7=tool name: 'Maven 3.3.9 (Windows)', type: 'hudson.tasks.Maven$MavenInstallation'
            dir(MAVEN_WIN_J7) {
                MAVEN_WIN_J7=pwd()
            }
            echo "Driving integration tests using Maven $MAVEN_WIN_J7"
            def JAVA_WIN_J7=tool name: 'JDK 1.7 (unlimited security) 64-bit Windows only', type: 'hudson.model.JDK'
            dir(JAVA_WIN_J7) {
                JAVA_WIN_J7=pwd()
            }
            echo "Running integration tests with Java $JAVA_WIN_J7"
            dir('test') {
                def WORK_DIR=pwd()
                git(url:'https://git-wip-us.apache.org/repos/asf/maven-integration-testing.git', branch: itBranch)
                bat "if exist it-local-repo rmdir /s /q it-local-repo"
                bat "if exist apache-maven-dist.zip del /q apache-maven-dist.zip"
                withEnv(["Path+MAVEN=$MAVEN_WIN_J7\\bin","Path+JDK=$JAVA_WIN_J7\\bin","JAVA_HOME=$JAVA_WIN_J7"]) {
                    bat "set"
                    unstash 'dist'
                    bat "mvn clean install -Prun-its -B -U -V -Dmaven.test.failure.ignore=true -Dmaven.repo.local=$WORK_DIR/it-local-repo -DmavenDistro=$WORK_DIR/apache-maven-dist.zip"
                }
                junit allowEmptyResults: true, testResults:'core-it-support/**/target/*-reports/*.xml,core-it-suite/target/*-reports/*.xml'
            }
        }
    }, winJava8: {
        node('Windows') {
            def MAVEN_WIN_J8=tool name: 'Maven 3.3.9 (Windows)', type: 'hudson.tasks.Maven$MavenInstallation'
            dir(MAVEN_WIN_J8) {
                MAVEN_WIN_J8=pwd()
            }
            echo "Driving integration tests using Maven $MAVEN_WIN_J8"
            def JAVA_WIN_J8=tool name: 'JDK 1.8 (unlimited security) 64-bit Windows only', type: 'hudson.model.JDK'
            dir(JAVA_WIN_J8) {
                JAVA_WIN_J8=pwd()
            }
            echo "Running integration tests with Java $JAVA_WIN_J8"
            dir('test') {
                def WORK_DIR=pwd()
                git(url:'https://git-wip-us.apache.org/repos/asf/maven-integration-testing.git', branch: itBranch)
                bat "if exist it-local-repo rmdir /s /q it-local-repo"
                bat "if exist apache-maven-dist.zip del /q apache-maven-dist.zip"
                withEnv(["Path+MAVEN=$MAVEN_WIN_J8\\bin","Path+JDK=$JAVA_WIN_J8\\bin","JAVA_HOME=$JAVA_WIN_J8"]) {
                    bat "set"
                    unstash 'dist'
                    bat "mvn clean install -Prun-its -B -U -V -Dmaven.test.failure.ignore=true -Dmaven.repo.local=$WORK_DIR/it-local-repo -DmavenDistro=$WORK_DIR/apache-maven-dist.zip"
                }
                junit allowEmptyResults: true, testResults:'core-it-support/**/target/*-reports/*.xml,core-it-suite/target/*-reports/*.xml'
            }
        }
    }
} finally {
    node('ubuntu') {
        emailext body: "See ${env.BUILD_URL}", recipientProviders: [[$class: 'CulpritsRecipientProvider'], [$class: 'FailingTestSuspectsRecipientProvider'], [$class: 'FirstFailingBuildSuspectsRecipientProvider']], replyTo: 'dev@maven.apache.org', subject: "Maven Jenkinsfile finished with ${currentBuild.result}", to: 'notifications@maven.apache.org'
    }
}

