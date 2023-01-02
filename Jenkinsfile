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
                    // when {
                    //     anyOf {
                    //         expression {params.ENVIRONMENT == 'all'}
                    //         expression {params.ENVIRONMENT == 'Router'}
                    //     }
                    // }
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