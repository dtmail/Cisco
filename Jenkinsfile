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
            }                        
        }                
    }                
}
