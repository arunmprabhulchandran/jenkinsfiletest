def maintenanceON() {
    sh '''#!/bin/bash
            set -x
            sudo sed -i '3s/set $maintenance off/set $maintenance on/g' /usr/local/nginx/includes/maintenance.conf
            sudo service nginx reload
    '''
}
def maintenanceOFF() {
    sh '''#!/bin/bash
            set -x
            sudo sed -i '3s/set $maintenance on/set $maintenance off/g' /usr/local/nginx/includes/maintenance.conf
            sudo service nginx reload    '''
}

def ShutdownServer() {
    sh '''#!/bin/bash
            set -x
            sudo docker exec --user="flash" appserver1 /bin/bash -c -l "./home/flash/script/build_stopserver.sh"
    '''
}

def startupServer() {
    sh '''#!/bin/bash
            set -x
            sudo docker exec --user="flash" appserver1 /bin/bash -c -l "./home/flash/script/app_start.sh"
    '''
}

def BuildWithLoadDefault() {
    sh '''#!/bin/bash
            set -x
            sudo docker exec --user="flash" appserver1 /bin/bash -c -l "./home/flash/script/build_withLoadDefault.sh"
    '''
}

def BuildWithoutLoadDefault() {
    sh '''#!/bin/bash
            set -x
            sudo docker exec --user="flash" appserver1 /bin/bash -c -l "./home/flash/script/build_withoutLoadDefault.sh"
    '''
}

def ExprotWarFile() {
    sh '''#!/bin/bash
            set -x
            cd /home/letslocalise/.jenkins/workspace/DEV_parameterized_scripted_pipeline/ 
            node -v
            npm -v
            gulp
            ./gradlew exportTomcatApps -PexportPath=/apps/dev_war_export
            cp -r /apps/dev_war_export/vc20/ /opt/appserver/letslocalise
            mkdir -p /opt/appserver/letslocalise/etc/tomcat/DEVInstance/logs
            touch /opt/appserver/letslocalise/etc/tomcat/DEVInstance/logs/catalina.out
            mkdir -p /opt/appserver/letslocalise/etc/tomcat/DEVInstance/temp
    '''
}

def RestoreSoftwareAndConfig() {
        sh '''#!/bin/bash
            set -x
            sudo docker exec --user="flash" appserver1 /bin/bash -c -l "cp -r /opt/appserver/script /home/flash/;./home/flash/script/restore_software_config.sh"
        '''
}
def ReplaceLLcoreschema() {
        sh '''#!/bin/bash
            set -x
            sudo docker exec --user="flash" appserver1 /bin/bash -c -l "cp -r /opt/appserver/script /home/flash/;./home/flash/script/llcore_cp.sh"
            mv /opt/appserver/llcore /opt/solr/
            sudo docker exec --user="flash" solr /bin/bash -c -l "cp -r /opt/solr/script /home/flash/;./home/flash/script/llcore.sh"
            rm -r /opt/solr/llcore
        '''
}
def BackupAndRestoreDocker() {
    sh '''#!/bin/bash
            set -x
            #mv /opt/appserver/warfile/* /apps/archive/warfile/
            #mv /opt/appserver/app_logs/* /apps/logs/
            sudo docker-compose --compatibility -f /apps/script/docker-compose-dev.yml down
            cd /opt/application_logs
            cp catalina.out catalina.$(date '+%Y-%m-%d-%s').log
            sudo docker-compose --compatibility -f /apps/script/docker-compose-dev.yml up -d
            rm -rf /opt/appserver/letslocalise/
    '''
}

//properties([[$class: 'JiraProjectProperty'], parameters([choice(choices: 'sprint_6\nsprint_7\nsprint_8\nll_newUI', description: 'Select branch to Build', name: 'Branch')])])
//properties([[$class: 'JiraProjectProperty'], parameters([choice1(choices: ['No', 'Yes'], description: '', name: 'loadDefault')])])
//properties([[$class: 'JiraProjectProperty'], parameters([choice2(choices: ['No', 'Yes'], description: '', name: 'LLcore')])])
//properties([
//  parameters([
//        choice(choices: '2021_sprint2.5\n2021_sprint2\n2021_sprint1.5\n2021_sprint1\nq2sprint8_hotfix\nq2sprint8\nq2sprint7_hotfix\nq2sprint7\nq2delta_hotfix\nq2sprint6_delta\nq2sprint6_hotfix\nq2sprint6\nq2sprint5_hotfix\nq2sprint5\nq2sprint4_hotfix\nq2sprint4\nq2sprint3_hotfix\nq2sprint3\nq2sprint2_hotfix\nq2sprint2\nQ2Sprint1\nsprint_9\nsprint_9_about_us\nsprint_8_hotfix\nsprint_8\nsprint_7\nsprint_6\nll_newUI\nsummer_fair_new', description: 'Select branch to Build', name: 'Branch'),
//        choice(choices: 'No\nYes', , name: 'loadDefault'),
//        choice(choices: 'No\nYes', name: 'LLcore')
//  ])
//])
properties([
    parameters([
        gitParameter(branch: '',
                     branchFilter: 'origin/(.*)',
                     defaultValue: 'master',
                     description: '',
                     name: 'BRANCH',
                     quickFilterEnabled: false,
                     selectedValue: 'NONE',
                     sortMode: 'NONE',
                     tagFilter: '*',
                     type: 'PT_BRANCH'),
        choice(choices: 'No\nYes', , name: 'loadDefault'),
        choice(choices: 'No\nYes', name: 'LLcore')
                     
    ])
])
node {
    stage('maintenance ON')
        {
            echo 'Set Maintenance on'
            maintenanceON()
        }
    stage('Checkout') {
            echo "Pulling changes from ${params.BRANCH}"
            //git branch: 'sprint_6', credentialsId: '73953242-bdc8-49b1-a484-a6d70c050916', url: 'https://saiprasadvks@gitlab.com/vastpro-group/letslocalise.git'
            //checkout([$class: 'GitSCM', branches: [[name: '${params.Branch}']], browser: [$class: 'GitLab', repoUrl: ''], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '73953242-bdc8-49b1-a484-a6d70c050916', url: 'https://saiprasadvks@gitlab.com/vastpro-group/letslocalise.git']]])
            git(url: 'https://gitlab.com/letslocalisecouk/letslocalise.git', branch: "${params.BRANCH}", credentialsId: 'thana-gitlab', changelog: true)
    }
    stage('Shutdown Server') {
        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE')
        {
            echo "Shutdown server"
            ShutdownServer()
        }
    }
    stage('Backup Logs,war file  & Restore Docker') {
            echo "Backup logs"
            BackupAndRestoreDocker()
    }
    stage ('Assemble EAR') 
    {
            echo "Assemble EAR"
            ExprotWarFile()
    }
    stage('Restore Docker Software and Config ')
        {
            echo "Restore Config"
            RestoreSoftwareAndConfig()
        }
    if( params.loadDefault == "Yes" ) {
        stage('BuildWithLoadDefault') {
            echo "${params.loadDefault}"
            BuildWithLoadDefault()
        }
    }else if( params.loadDefault == "No" ) { 
        stage('BuildWithoutLoadDefault') {
            echo "${params.loadDefault}"
            BuildWithoutLoadDefault()
         }
    }
    if( params.LLcore == "Yes" ) {
        stage('Replace LLCore schema') {
            echo "${params.LLcore}"
            ReplaceLLcoreschema()
        }
    }
    stage('Start Server')
        {
            echo 'Starting server'
            startupServer()
        }
    stage('maintenance OFF')
        {
            echo 'Set Maintenance on'
            maintenanceOFF()
        }
}
