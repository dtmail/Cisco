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

pipeline {
    agent any
    stages {
        stage ('Build and run'){
            parallel {
                stage('DEV') {
                    stages {
                        stage('Hello') {
                            steps {
                                echo 'Hello World'
                            }
                        }

                        stage('Build') {
                            steps {
                                echo 'Building'
                            }
                        }

                        stage('Deploy') {
                            steps {
                                echo 'Deploying'
                            }
                        }

                        stage('Test') {
                            steps {
                                echo 'Testing'
                            }
                        }

                        stage('Release') {
                            steps {
                                echo 'Releasing'
                            }
                        }

                    }
                }
                stage('NON-PROD') {
                    stages {
                        stage('Hello') {
                            steps {
                                echo 'Hello World'
                            }
                        }

                        stage('Build') {
                            steps {
                                echo 'Building'
                            }
                        }

                        stage('Deploy') {
                            steps {
                                echo 'Deploying'
                            }
                        }

                        stage('Test') {
                            steps {
                                echo 'Testing'
                            }
                        }

                        stage('Release') {
                            steps {
                                echo 'Releasing'
                            }
                        }

                    }
                }
                stage('PROD') {
                    when {
                        anyOf {
                            // expression {params.ENVIRONMENT == 'all'}
                            expression {params.ENVIRONMENT == 'Router'}
                        }
                    }
                    stages {
                        stage('Hello') {
                            // when {
                            //     not {
                            //         branch "main"
                            //     }
                            // }
                            steps {
                                echo 'Hello World'
                            }
                        }

                        stage('Build') {
                            steps {
                                echo 'Building'
                            }
                        }

                        stage('Deploy') {
                            steps {
                                echo 'Deploying'
                            }
                        }

                        stage('Test') {
                            steps {
                                echo 'Testing'
                            }
                        }

                        stage('Release') {
                            steps {
                                echo 'Releasing'
                            }
                        }

                    }
                }                       
            }                        
        }
        stage('Cleanup'){
            steps {
                echo 'Cleanup'
            }
        }
    }                
}