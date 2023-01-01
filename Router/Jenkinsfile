@Library(['cop-pipeline-step']) _
properties([
    parameters([
        choice(
            choices: ['all','non-prod','prod','dev'],
            description: '',
            name: 'ENVIRONMENT',
            defaultValue: 'all'
        ),
        string(
            name: 'SPECIFIC_HOSTS', 
            description: 'Specified one or multiple host(s) in the selected ENVIROMENT to run the pipeline against.<br/>Multiple hosts must be devided by <b>","</b> e.g. <b>"test_device1,test_device2"</b>'
        ),
        booleanParam(
            defaultValue: false, 
            description: '', 
            name: 'ENABLE_DEBUG'
        ),
        booleanParam(
            defaultValue: false, 
            description: '', 
            name: 'DRY_RUN'
        ),
        booleanParam(
            defaultValue: false, 
            description: '', 
            name: 'IGNORE_STATE'
        ),
    ])
])

script {
  switch(params.ENVIRONMENT) {
    case 'non-prod':
        INVENTORY_FILE = "./inventory/"+params.ENVIRONMENT+"_host.yml"
        break
    case 'prod':
        INVENTORY_FILE = "./inventory/"+params.ENVIRONMENT+"_host.yml"
        break
    case 'dev':
        INVENTORY_FILE = "./inventory/"+params.ENVIRONMENT+"_host.yml"
        break
  }
}

config = [
    notify: [
        slack: [
            onCondition: ['Failure', 'Success'],
            channel: "#tgw-dev-notifications",
        ]
    ],
    aws: [
        role  : 'Networking.Cicd',
        account: '471655526001',
        region   : 'us-west-2',
    ],
    s3: [
        bucket: 'network-hub-infrastructure',
        path: '_statefile.yml',
        file: '_statefile.yml',
    ]
]
pipeline {
    agent any
    stages {
        stage ('Build and run'){
            parallel {
                stage('DEV') {
                    when { 
                        anyOf {
                            expression { params.ENVIRONMENT == 'all' }
                            expression { params.ENVIRONMENT == 'dev' }
                        }
                    }
                    stages {
                        stage('Download statefile from S3') {
                            steps {
                                withAWS(role: config.aws.role, roleAccount: config.aws.account, region: config.aws.region){
                                    s3Download(file: 'dev'+config.s3.file, path: 'dev'+config.s3.path, bucket: config.s3.bucket, force:true)
                                }
                            }
                        }
                        stage('Dry-run') {
                            when {
                                not {
                                    branch "main"
                                }
                            }                            
                            steps {
                                genericBuild(
                                    dockerfile: 'Dockerfile',
                                    dockerfileImageName: 'network-hub-infra-docker',
                                    cmd: 'pip install -U -r requirements.txt',
                                    cerberus: [
                                        sdbPath: 'app/ndocore/network-hub-build',
                                        sdbKeys: [
                                            "tacacs_username" : "TACACS_USERNAME",
                                            "tacacs_password" : "TACACS_PASSWORD",
                                            "last_resort_username" : "LAST_RESORT_USERNAME",
                                            "last_resort_password" : "LAST_RESORT_PASSWORD",
                                            "snmp_auth_pwd": "SNMP_AUTH_PWD",
                                            "snmp_priv_pwd": "SNMP_PRIV_PWD",
                                            "tacacs_key": "TACACS_KEY",
                                            "snmp_auth_pwd_nexus": "SNMP_AUTH_PWD_NEXUS",
                                            "snmp_priv_pwd_nexus": "SNMP_PRIV_PWD_NEXUS",
                                            "tacacs_key_nexus": "TACACS_KEY_NEXUS",
                                            "tacacs_key_iosxe": "TACACS_KEY_IOSXE",
                                            "snmp_auth_pwd_iosxe": "SNMP_AUTH_PWD_IOSXE",
                                            "snmp_priv_pwd_iosxe": "SNMP_PRIV_PWD_IOSXE",
                                            "last_resort_password_hash": "LAST_RESORT_PWD_HASH",
                                            "last_resort_password_hash7": "LAST_RESORT_PWD_HASH7"
                                        ]
                                    ],
                                    envVars: [ 

                                        "INVENTORY=./inventory/dev_host.yml",
                                        "GWAN_STATEFILE=dev_statefile.yml",
                                        "DRYRUN=True",
                                        "ENVIRONMENT=dev",
                                        "NOT_SEND_SLACK_MSG=True",
                                        "CHECK_VAR_GITHUB=True",
                                        "BUILD_URL=$env.BUILD_URL",
                                        "BUILD_NAME=$currentBuild.fullDisplayName",
                                        "DEBUG=True"
                                    ],
                                    withClosure: {
                                        sh "python3 network_build.py" 
                                    }
                                )
                            }
                        }
                        stage('Deploying'){
                             when {
                                branch "main"
                            }
                            steps {
                                script {
                                    if (params.ENABLE_DEBUG) {
                                        env.DEBUG = "True"
                                    }
                                    if (params.IGNORE_STATE){
                                        env.IGNORESTATE = "True"
                                    }
                                    if (params.SPECIFIC_HOSTS){
                                        env.HOSTS = SPECIFIC_HOSTS
                                    }
                                }
                                genericBuild(
                                    dockerfile: 'Dockerfile',
                                    dockerfileImageName: 'network-hub-infra-docker',
                                    cmd: 'pip install -U -r requirements.txt',
                                    cerberus: [
                                        sdbPath: 'app/ndocore/network-hub-build',
                                        sdbKeys: [
                                            "tacacs_username" : "TACACS_USERNAME",
                                            "tacacs_password" : "TACACS_PASSWORD",
                                            "last_resort_username" : "LAST_RESORT_USERNAME",
                                            "last_resort_password" : "LAST_RESORT_PASSWORD",
                                            "snmp_auth_pwd": "SNMP_AUTH_PWD",
                                            "snmp_priv_pwd": "SNMP_PRIV_PWD",
                                            "tacacs_key": "TACACS_KEY",
                                            "snmp_auth_pwd_nexus": "SNMP_AUTH_PWD_NEXUS",
                                            "snmp_priv_pwd_nexus": "SNMP_PRIV_PWD_NEXUS",
                                            "tacacs_key_nexus": "TACACS_KEY_NEXUS",
                                            "tacacs_key_iosxe": "TACACS_KEY_IOSXE",
                                            "snmp_auth_pwd_iosxe": "SNMP_AUTH_PWD_IOSXE",
                                            "snmp_priv_pwd_iosxe": "SNMP_PRIV_PWD_IOSXE",
                                            "last_resort_password_hash": "LAST_RESORT_PWD_HASH",
                                            "last_resort_password_hash7": "LAST_RESORT_PWD_HASH7"
                                        ]
                                    ],
                                    envVars: [ 

                                        "INVENTORY=./inventory/dev_host.yml",
                                        "GWAN_STATEFILE=dev_statefile.yml",
                                        "ENVIRONMENT=dev",
                                        "DEBUG=True"
                                    ],
                                    withClosure: {
                                        sh "python3 network_build.py"
                                    }
                                )
                            }
                        }
                        stage('Updating SolarWinds'){
                             when {
                                branch "main"
                            }
                            steps {
                                script {
                                    if (params.ENABLE_DEBUG) {
                                        env.DEBUG = "True"
                                    }
                                }
                                genericBuild(
                                    dockerfile: 'Dockerfile',
                                    dockerfileImageName: 'network-hub-infra-docker',
                                    cmd: 'pip install -U -r requirements.txt',
                                    cerberus: [
                                        sdbPath: 'app/ndocore/network-hub-build',
                                        sdbKeys: [
                                            "SW_USERNAME" : "SW_USERNAME",
                                            "SW_PASSWORD" : "SW_PASSWORD",

                                        ]
                                    ],
                                    envVars: [ 

                                        "INVENTORY=./inventory/dev_host.yml",
                                        "GWAN_STATEFILE=dev_statefile.yml",
                                        "ENVIRONMENT=dev"
                                    ],
                                    withClosure: {
                                        sh "python3 solarwinds_update.py"
                                    }
                                )
                            }
                        }
                        stage('Upload statefile to S3') {
                            when {
                                branch "main" 
                            }
                            steps {
                                withAWS(role: config.aws.role, roleAccount: config.aws.account, region: config.aws.region){

                                    s3Upload(file:'dev'+config.s3.file, path: 'dev'+config.s3.path, bucket: config.s3.bucket)

                                }
                            }
                        }
                    }
                    post {
                        failure {
                            notifyOnCondition(
                                [
                                    action : 'notify',
                                    slack  : [
                                        message: "*<${env.BUILD_URL}|${currentBuild.fullDisplayName} - FAILURE DEV>*",
                                        color  : "danger"
                                    ] + config.notify.slack
                                ],
                                currentBuild.currentResult,
                            )
                        }
                        success {
                            notifyOnCondition(
                                [
                                    action : 'notify',
                                    slack  : [
                                        message: "*<${env.BUILD_URL}|${currentBuild.fullDisplayName} - SUCCESS DEV>*",
                                        color  : "good"
                                    ] + config.notify.slack
                                ],
                                currentBuild.currentResult,
                            )
                        }
                    }
                }
                stage('NON-PROD') {
                    when { 
                        anyOf {
                            expression { params.ENVIRONMENT == 'all' }
                            expression { params.ENVIRONMENT == 'non-prod' }
                        }
                    }
                    stages {
                        stage('Download statefile from S3') {
                            steps {
                                withAWS(role: config.aws.role, roleAccount: config.aws.account, region: config.aws.region){
                                    s3Download(file: 'non-prod'+config.s3.file, path: 'non-prod'+config.s3.path, bucket: config.s3.bucket, force:true)
                                }
                            }
                        }
                        stage('Dry-run') {
                            when {
                                not {
                                    branch "main"
                                }
                            }
                            steps {
                                genericBuild(
                                    dockerfile: 'Dockerfile',
                                    dockerfileImageName: 'network-hub-infra-docker',
                                    cmd: 'pip install -U -r requirements.txt',
                                    cerberus: [
                                        sdbPath: 'app/ndocore/network-hub-build',
                                        sdbKeys: [
                                            "netconf_username" : "TACACS_USERNAME",
                                            "netconf_password" : "TACACS_PASSWORD",
                                            "last_resort_username" : "LAST_RESORT_USERNAME",
                                            "last_resort_password" : "LAST_RESORT_PASSWORD",
                                            "snmp_auth_pwd": "SNMP_AUTH_PWD",
                                            "snmp_priv_pwd": "SNMP_PRIV_PWD",
                                            "tacacs_key": "TACACS_KEY",
                                            "snmp_auth_pwd_nexus": "SNMP_AUTH_PWD_NEXUS",
                                            "snmp_priv_pwd_nexus": "SNMP_PRIV_PWD_NEXUS",
                                            "tacacs_key_nexus": "TACACS_KEY_NEXUS",
                                            "tacacs_key_iosxe": "TACACS_KEY_IOSXE",
                                            "snmp_auth_pwd_iosxe": "SNMP_AUTH_PWD_IOSXE",
                                            "snmp_priv_pwd_iosxe": "SNMP_PRIV_PWD_IOSXE",
                                            "last_resort_password_hash": "LAST_RESORT_PWD_HASH",
                                            "last_resort_password_hash7": "LAST_RESORT_PWD_HASH7"
                                        ]
                                    ],
                                    envVars: [ 
                                        "INVENTORY=./inventory/non-prod_host.yml",
                                        "GWAN_STATEFILE=non-prod_statefile.yml",
                                        "DRYRUN=True",
                                        "ENVIRONMENT=non-prod",
                                        "DEBUG=True"
                                    ],
                                    withClosure: {
                                        sh "python3 network_build_non-prod.py" 
                                    }
                                )
                            }
                        }
                        stage('Deploying'){
                            when {
                                branch "main"
                            }
                            steps {
                                script {
                                    if (params.ENABLE_DEBUG) {
                                        env.DEBUG = "True"
                                    }
                                    if (params.IGNORE_STATE){
                                        env.IGNORESTATE = "True"
                                    }
                                    if (params.SPECIFIC_HOSTS){
                                        env.HOSTS = SPECIFIC_HOSTS
                                    }
                                }
                                genericBuild(
                                    dockerfile: 'Dockerfile',
                                    dockerfileImageName: 'network-hub-infra-docker',
                                    cmd: 'pip install -U -r requirements.txt',
                                    cerberus: [
                                        sdbPath: 'app/ndocore/network-hub-build',
                                        sdbKeys: [
                                            "netconf_username" : "TACACS_USERNAME",
                                            "netconf_password" : "TACACS_PASSWORD",
                                            "last_resort_username" : "LAST_RESORT_USERNAME",
                                            "last_resort_password" : "LAST_RESORT_PASSWORD",
                                            "snmp_auth_pwd": "SNMP_AUTH_PWD",
                                            "snmp_priv_pwd": "SNMP_PRIV_PWD",
                                            "tacacs_key": "TACACS_KEY",
                                            "snmp_auth_pwd_nexus": "SNMP_AUTH_PWD_NEXUS",
                                            "snmp_priv_pwd_nexus": "SNMP_PRIV_PWD_NEXUS",
                                            "tacacs_key_nexus": "TACACS_KEY_NEXUS",
                                            "tacacs_key_iosxe": "TACACS_KEY_IOSXE",
                                            "snmp_auth_pwd_iosxe": "SNMP_AUTH_PWD_IOSXE",
                                            "snmp_priv_pwd_iosxe": "SNMP_PRIV_PWD_IOSXE",
                                            "last_resort_password_hash": "LAST_RESORT_PWD_HASH",
                                            "last_resort_password_hash7": "LAST_RESORT_PWD_HASH7"
                                        ]
                                    ],
                                    envVars: [ 
                                        "INVENTORY=./inventory/non-prod_host.yml",
                                        "GWAN_STATEFILE=non-prod_statefile.yml",
                                        "ENVIRONMENT=non-prod" 
                                    ],
                                    withClosure: {
                                        sh "python3 network_build_non-prod.py" 
                                    }
                                )
                            }
                        }
                        stage('Upload statefile to S3') {
                            when {
                                branch "main" 
                            }
                            steps {
                                withAWS(role: config.aws.role, roleAccount: config.aws.account, region: config.aws.region){
                                    s3Upload(file:'non-prod'+config.s3.file, path: 'non-prod'+config.s3.path, bucket: config.s3.bucket)
                                }
                            }
                        }
                    }
                    post {
                        failure {
                            notifyOnCondition(
                                [
                                    action : 'notify',
                                    slack  : [
                                        message: "*<${env.BUILD_URL}|${currentBuild.fullDisplayName} - FAILURE NON-PROD>*",
                                        color  : "danger"
                                    ] + config.notify.slack
                                ],
                                currentBuild.currentResult,
                            )
                        }
                        success {
                            notifyOnCondition(
                                [
                                    action : 'notify',
                                    slack  : [
                                        message: "*<${env.BUILD_URL}|${currentBuild.fullDisplayName} - SUCCESS NON-PROD>*",
                                        color  : "good"
                                    ] + config.notify.slack
                                ],
                                currentBuild.currentResult,
                            )
                        }
                    }
                }
                stage('PROD') {
                    when { 
                        anyOf {
                            expression { params.ENVIRONMENT == 'all' }
                            expression { params.ENVIRONMENT == 'prod' }
                        }
                    }
                    stages {
                        stage('Download statefile from S3') {
                            steps {
                                withAWS(role: config.aws.role, roleAccount: config.aws.account, region: config.aws.region){
                                    s3Download(file: 'prod'+config.s3.file, path: 'prod'+config.s3.path, bucket: config.s3.bucket, force:true)
                                }
                            }
                        }
                        stage('Dry-run') {
                            when {
                                not {
                                    branch "main"
                                }
                            }
                            steps {
                                genericBuild(
                                    dockerfile: 'Dockerfile',
                                    dockerfileImageName: 'network-hub-infra-docker',
                                    cmd: 'pip install -U -r requirements.txt',
                                    cerberus: [
                                        sdbPath: 'app/ndocore/network-hub-build',
                                        sdbKeys: [
                                            "tacacs_username" : "TACACS_USERNAME",
                                            "tacacs_password" : "TACACS_PASSWORD",
                                            "last_resort_username" : "LAST_RESORT_USERNAME",
                                            "last_resort_password" : "LAST_RESORT_PASSWORD",
                                            "snmp_auth_pwd": "SNMP_AUTH_PWD",
                                            "snmp_priv_pwd": "SNMP_PRIV_PWD",
                                            "tacacs_key": "TACACS_KEY",
                                            "snmp_auth_pwd_nexus": "SNMP_AUTH_PWD_NEXUS",
                                            "snmp_priv_pwd_nexus": "SNMP_PRIV_PWD_NEXUS",
                                            "tacacs_key_nexus": "TACACS_KEY_NEXUS",
                                            "tacacs_key_iosxe": "TACACS_KEY_IOSXE",
                                            "snmp_auth_pwd_iosxe": "SNMP_AUTH_PWD_IOSXE",
                                            "snmp_priv_pwd_iosxe": "SNMP_PRIV_PWD_IOSXE",
                                            "last_resort_password_hash": "LAST_RESORT_PWD_HASH",
                                            "last_resort_password_hash7": "LAST_RESORT_PWD_HASH7"
                                        ]
                                    ],
                                    envVars: [ 

                                        "INVENTORY=./inventory/prod_host.yml",
                                        "GWAN_STATEFILE=prod_statefile.yml",
                                        "DRYRUN=True",
                                        "ENVIRONMENT=prod",
                                        "NOT_SEND_SLACK_MSG=True",
                                        "CHECK_VAR_GITHUB=True",
                                        "BUILD_URL=$env.BUILD_URL",
                                        "BUILD_NAME=$currentBuild.fullDisplayName",
                                        "DEBUG=True"
                                    ],
                                    withClosure: {
                                        sh "python3 network_build.py"
                                    }
                                )
                            }
                        }
                        stage('Deploying'){
                             when {
                                branch "main"
                            }
                            steps {
                                script {
                                    if (params.ENABLE_DEBUG) {
                                        env.DEBUG = "True"
                                    }
                                    if (params.IGNORE_STATE){
                                        env.IGNORESTATE = "True"
                                    }
                                    if (params.SPECIFIC_HOSTS){
                                        env.HOSTS = SPECIFIC_HOSTS
                                    }
                                }
                                genericBuild(
                                    dockerfile: 'Dockerfile',
                                    dockerfileImageName: 'network-hub-infra-docker',
                                    cmd: 'pip install -U -r requirements.txt',
                                    cerberus: [
                                        sdbPath: 'app/ndocore/network-hub-build',
                                        sdbKeys: [
                                            "tacacs_username" : "TACACS_USERNAME",
                                            "tacacs_password" : "TACACS_PASSWORD",
                                            "last_resort_username" : "LAST_RESORT_USERNAME",
                                            "last_resort_password" : "LAST_RESORT_PASSWORD",
                                            "snmp_auth_pwd": "SNMP_AUTH_PWD",
                                            "snmp_priv_pwd": "SNMP_PRIV_PWD",
                                            "tacacs_key": "TACACS_KEY",
                                            "snmp_auth_pwd_nexus": "SNMP_AUTH_PWD_NEXUS",
                                            "snmp_priv_pwd_nexus": "SNMP_PRIV_PWD_NEXUS",
                                            "tacacs_key_nexus": "TACACS_KEY_NEXUS",
                                            "tacacs_key_iosxe": "TACACS_KEY_IOSXE",
                                            "snmp_auth_pwd_iosxe": "SNMP_AUTH_PWD_IOSXE",
                                            "snmp_priv_pwd_iosxe": "SNMP_PRIV_PWD_IOSXE",
                                            "last_resort_password_hash": "LAST_RESORT_PWD_HASH",
                                            "last_resort_password_hash7": "LAST_RESORT_PWD_HASH7"
                                        ]
                                    ],
                                    envVars: [ 

                                        "INVENTORY=./inventory/prod_host.yml",
                                        "GWAN_STATEFILE=prod_statefile.yml",
                                        "ENVIRONMENT=prod",
                                        "DEBUG=True"
                                    ],
                                    withClosure: {
                                        sh "python3 network_build.py" 
                                    }
                                )
                            }
                        }
                        stage('Updating SolarWinds'){
                             when {
                                branch "main"
                            }
                            steps {
                                script {
                                    if (params.ENABLE_DEBUG) {
                                        env.DEBUG = "True"
                                    }
                                }
                                genericBuild(
                                    dockerfile: 'Dockerfile',
                                    dockerfileImageName: 'network-hub-infra-docker',
                                    cmd: 'pip install -U -r requirements.txt',
                                    cerberus: [
                                        sdbPath: 'app/ndocore/network-hub-build',
                                        sdbKeys: [
                                            "SW_USERNAME" : "SW_USERNAME",
                                            "SW_PASSWORD" : "SW_PASSWORD",

                                        ]
                                    ],
                                    envVars: [ 

                                        "INVENTORY=./inventory/prod_host.yml",
                                        "GWAN_STATEFILE=prod_statefile.yml",
                                        "ENVIRONMENT=prod"
                                    ],
                                    withClosure: {
                                        sh "python3 solarwinds_update.py" 
                                    }
                                )
                            }
                        }
                        stage('Updating DNS') {
                            when {
                                branch "main"
                            }
                            steps {
                                genericBuild(
                                    dockerfile: 'Dockerfile',
                                    dockerfileImageName: 'network-hub-infra-docker',
                                    cmd: 'pip install -U -r requirements.txt',
                                    cerberus: [
                                        sdbPath: 'app/ndocore/network-hub-build',
                                        sdbKeys: [
                                            "OKTA_CLIENT_SECRET" : "OKTA_CLIENT_SECRET",
                                            "OKTA_CLIENT_SECRET_SANDBOX" : "OKTA_CLIENT_SECRET_SANDBOX",
                                            "OKTA_CLIENT_ID" : "OKTA_CLIENT_ID",
                                            "DDI_URL_PROD" : "DDI_URL"
                                        ]
                                    ],
                                    envVars: [ 
                                        "INVENTORY=./inventory/prod_host.yml",
                                        "GWAN_STATEFILE=prod_statefile.yml",
                                        "JUST_DO_IT=True"
                                    ],
                                    withClosure: {
                                        sh "python3 dns_ddi_update.py" 
                                    }
                                )
                            }
                        }
                        stage('Upload statefile to S3') {
                            when {
                                branch "main" 
                            }
                            steps {
                                withAWS(role: config.aws.role, roleAccount: config.aws.account, region: config.aws.region){

                                    s3Upload(file:'prod'+config.s3.file, path: 'prod'+config.s3.path, bucket: config.s3.bucket)

                                }
                            }
                        }
                    }
                    post {
                        failure {
                            notifyOnCondition(
                                [
                                    action : 'notify',
                                    slack  : [
                                        message: "*<${env.BUILD_URL}|${currentBuild.fullDisplayName} - FAILURE PROD>*",
                                        color  : "danger"
                                    ] + config.notify.slack
                                ],
                                currentBuild.currentResult,
                            )
                        }
                        success {
                            notifyOnCondition(
                                [
                                    action : 'notify',
                                    slack  : [
                                        message: "*<${env.BUILD_URL}|${currentBuild.fullDisplayName} - SUCCESS PROD>*",
                                        color  : "good"
                                    ] + config.notify.slack
                                ],
                                currentBuild.currentResult,
                            )
                        }
                    }
                }
            }
        }                
        stage('Cleanup'){
            steps {
                cleanWs()
            }
        }
    }
}
