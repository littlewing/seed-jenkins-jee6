pipeline {
    agent any
    parameters {
        booleanParam(name: 'IS_RELEASE', defaultValue: false, description: 'Voulez vous faire une release')
        string(name: 'SVN_URL', defaultValue: 'http://svnhm/svn/eand_src/attitudes/attitudes/trunk/', description: 'URL REPOSITORY SVN')
    }
    stages {
        stage('Init') {
            steps {
                script {
                    timeout(time: 1, unit: 'HOURS') {
                        if (params.IS_RELEASE == true) {
                            def release = input id: 'release', message: 'test', parameters: [string(defaultValue: '1.0.0-SNAPSHOT', description: 'New SNAPSHOT', name: 'developmentVersion'), string(defaultValue: '1.0.0_01', description: 'New release', name: 'releaseVersion')], submitterParameter: 'submitter'
                            echo(release['developmentVersion'])
                        }
                    }
                    echo ">>Chargement du fichier jenkins-jee6 ..."
                    def jenkinsjee6 = fileLoader.load('src/main/groovy/jenkins-jee6')
                }
            }
        }
        stage('Checkout') {
            steps {
                checkout([$class: 'SubversionSCM', additionalCredentials: [], excludedCommitMessages: '', excludedRegions: '', excludedRevprop: '', excludedUsers: '', filterChangelog: false, ignoreDirPropChanges: false, includedRegions: '', locations: [[credentialsId: 'eandx00', depthOption: 'infinity', ignoreExternalsOption: true, local: '.', remote: params.SVNURL]], workspaceUpdater: [$class: 'UpdateUpdater']])
            }
        }
        stage('Build') {
            steps {
                withMaven(maven: 'maven-3.2') {
                    sh "mvn clean install -Dmaven.test.skip=true -P jdk8,int"
                }
            }
        }
        stage('Test') {
            steps {
                catchError {
                    withMaven(maven: 'maven-3.2') {
                        sh "mvn -fn test -P jdk8,int -Daggregate=true -DtestFailureIgnore=true"
                    }
                    junit "target/test-reports/TEST*.xml"
                }
                step([$class: 'Mailer', recipients: 'admin@somewhere'])
            }

        }
        stage('Deploy') {
            steps {
                echo "Deploiement"
                withMaven(maven: 'maven-3.2') {
                    sh "mvn deploy -Dmaven.test.skip=true -P jdk8,int"
                }
                script{
                    jenkinsjee6.deployInWeblogic()
                }
            }
        }

        stage('Release') {
            steps() {
                script {
                    if (params.IS_RELEASE == true) {
                        withMaven(maven: 'maven-3.2') {
                            sh "mvn release:prepare -Dmaven.test.skip=true -P jdk8,int"
                        }
                    }
                }


            }
        }
    }
}
