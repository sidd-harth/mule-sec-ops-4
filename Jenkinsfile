pipeline {
    agent any

    tools {
        maven 'M386'
    }

    environment {
        POM_GROUP_ID         = sh(script: "mvn help:evaluate -Dexpression='project.groupId' -q -DforceStdout", returnStdout: true).trim()
        POM_ARTIFACT_ID      = sh(script: "mvn help:evaluate -Dexpression='project.artifactId' -q -DforceStdout", returnStdout: true).trim()
        POM_VERSION          = sh(script: "mvn help:evaluate -Dexpression='project.version' -q -DforceStdout", returnStdout: true).trim()
        POM_MAJOR_VERSION    = sh(script: "mvn help:evaluate -Dexpression='project.version' -q -DforceStdout |  cut -d. -f1", returnStdout: true).trim()
        POM_MINOR_VERSION    = sh(script: "mvn help:evaluate -Dexpression='project.version' -q -DforceStdout |  cut -d. -f2", returnStdout: true).trim()
        POM_PATCH_VERSION    = sh(script: "mvn help:evaluate -Dexpression='project.version' -q -DforceStdout |  cut -d. -f3 | cut -d- -f1", returnStdout: true).trim()
        POM_MAVEN_QUALIFIER  = sh(script: "mvn help:evaluate -Dexpression='project.version' -q -DforceStdout | cut -d- -f2", returnStdout: true).trim()
        POM_SCM_URL          = sh(script: "mvn help:evaluate -Dexpression='project.scm.url' -q -DforceStdout", returnStdout: true).trim()
        MULE_RUNTIME_VERSION = sh(script: "mvn help:evaluate -Dexpression='app.runtime' -q -DforceStdout", returnStdout: true).trim()
        ANYPOINT_PLATFORM_CREDENTIALS = credentials('ANYPOINT_PLATFORM_CREDENTIALS')
        QA_CLIENT_ID = credentials('QA_CLIENT_ID')
        QA_CLIENT_SECRET = credentials('QA_CLIENT_SECRET')
        CONNECTED_APP_CLIENT_ID = credentials('CONNECTED_APP_CLIENT_ID')
        CONNECTED_APP_CLIENT_SECRET = credentials('CONNECTED_APP_CLIENT_SECRET')
    }

    parameters {
        //string( name: 'BRANCH', description: 'Name of the Release Branch to be built? (e.g. release/1.1.0-1)' )
        choice( name: 'ENVIRONMENT', choices: "qa\nprod", description: 'Environment where Mule Application will be deployed' )
        string( name: 'API_VERSION', defaultValue: '1.0.0', description: 'Version of API Instance for pairing with Mule Application (e.g. v1)' )
        string( name: 'ORGANIZATION', defaultValue: 'sec-org', description: 'Org or BusinessGroup to deploy the API' )
    }    
       
    stages {


        


    /*  stage('echo') {
            steps {
                echo "$ANYPOINT_PLATFORM_CREDENTIALS_USR"
                echo "$ANYPOINT_PLATFORM_CREDENTIALS_PSW"
                echo "${params.ORGANIZATION}"
                echo "${env.POM_ARTIFACT_ID}"
                echo "${params.API_VERSION}"
                echo "${env.MULE_RUNTIME_VERSION}"
                echo "${currentBuild.currentResult}"
            }
        }

        stage('Download from Artifactory') {
            steps {
                sh """ mvn dependency:copy -Dartifact="${env.POM_GROUP_ID}:${env.POM_ARTIFACT_ID}:${env.POM_VERSION}:jar:mule-application" """
            }
        }

        stage('Promote API to Testing') {
            when {
                expression { params.ENVIRONMENT == 'qa' }
            }
            steps {
                script {
                    echo "Promoting API from Development..."
                    sh """ newman run Promote-API.postman_collection-v7.json \
                                --env-var anypoint_username=$ANYPOINT_PLATFORM_CREDENTIALS_USR \
                                --env-var anypoint_password=$ANYPOINT_PLATFORM_CREDENTIALS_PSW \
                                --env-var anypoint_organisation=${params.ORGANIZATION} \
                                --env-var source_environment=dev \
                                --env-var target_environment=qa \
                                --env-var asset_id=${env.POM_ARTIFACT_ID} \
                                --env-var product_version=${params.API_VERSION} \
                                --env-var anypoint_runtime=${env.MULE_RUNTIME_VERSION} \
                                --disable-unicode \
                                --color on \
                                --reporters cli,json,htmlextra \
                                --reporter-htmlextra-export ./newman/promote-api-report.html \
                                --reporter-json-export promote-api-output.json """
                    echo "Promoted API from Testing: ${currentBuild.currentResult}"
                }
            }    
            post {
                success {
                    echo "...Promote API from Development Succeeded for ${env.BUILD_VERSION}: ${currentBuild.currentResult}"
                    publishHTML (target : [allowMissing: true,
                                            alwaysLinkToLastBuild: true,
                                            keepAll: true,
                                            reportDir: './newman',
                                            reportFiles: 'report.html',
                                            reportName: 'NewMan Promote Reports',
                                            reportTitles: 'Promote Report'])
                } 
                failure {
                    echo "...Promote API from Development Failed for ${env.BUILD_VERSION}: ${currentBuild.currentResult}"
                }
            }
        } 


        stage('autodiscovery') {
            steps {
                script {
                    def postman_envs = readJSON file: 'promote-api-output.json'
                    def auto_discovery_id = postman_envs.environment.values.findIndexOf{ it.key == "auto_api_id" }
                    def contract_client_id = postman_envs.environment.values.findIndexOf{ it.key == "client_id" }
                    def contract_client_secret = postman_envs.environment.values.findIndexOf{ it.key == "client_secret" }
                    print "Autodiscovery API ID: " + postman_envs.environment.values[auto_discovery_id].value
                    print "Contract Client ID: " + postman_envs.environment.values[contract_client_id].value
                    print "Contract Client Secret: " + postman_envs.environment.values[contract_client_secret].value
                }
            }
        }
        
        stage('Deploy to QA') {
            steps {
                script {
                    def postman_envs = readJSON file: 'promote-api-output.json'
                    def auto_discovery_id = postman_envs.environment.values.findIndexOf{ it.key == "auto_api_id" }
                   
                    sh """ mvn --batch-mode deploy -DmuleDeploy \
                                    -Denv=qa \
                                    -Dcloudhub.application.name=${env.POM_ARTIFACT_ID}-qa \
                                    -Dcloudhub.environment=qa \
                                    -Dartifact.path=target/dependency/${env.POM_ARTIFACT_ID}-${env.POM_VERSION}-mule-application.jar \
                                    -Dcloudhub.workers=1 \
                                    -Dcloudhub.worker.type=SMALL \
                                    -Dcloudhub.region=us-east-2 \
                                    -Dap.ca.client_id=$CONNECTED_APP_CLIENT_ID \
                                    -Dap.ca.client_secret=$CONNECTED_APP_CLIENT_SECRET \
                                    -Danypoint.platform.client.id=$QA_CLIENT_ID \
                                    -Danypoint.platform.client.secret=$QA_CLIENT_SECRET \
                                    -Dapi.id=${postman_envs.environment.values[auto_discovery_id].value} """ 
                }
            }
        }

        
        stage('QA Integration Testing') {
            steps {
                script {
                    def postman_envs = readJSON file: 'promote-api-output.json'
                    def client_id = postman_envs.environment.values.findIndexOf{ it.key == "client_id" }
                    def client_secret = postman_envs.environment.values.findIndexOf{ it.key == "client_secret" }
                    def environment_id = postman_envs.environment.values.findIndexOf{ it.key == "target_environment_id" }
                   
                    sh """ newman run Integration-Testing.postman_collection.json \
                                    --env-var anypoint_username=$ANYPOINT_PLATFORM_CREDENTIALS_USR \
                                    --env-var anypoint_password=$ANYPOINT_PLATFORM_CREDENTIALS_PSW \
                                    --env-var target_application_name=${env.POM_ARTIFACT_ID}-qa \
                                    --env-var client_id=${postman_envs.environment.values[client_id].value} \
                                    --env-var client_secret=${postman_envs.environment.values[client_secret].value} \
                                    --env-var target_environment_id=${postman_envs.environment.values[environment_id].value} \
                                    --env-var uriPath=hello \
                                    --disable-unicode \
                                    --color on \
                                    --reporters cli,json,htmlextra \
                                    --reporter-htmlextra-export ./newman/qa-integration-testing-report.html \
                                    --reporter-json-export qa-integration-testing-output.json """
                    echo "QA Integration Testing: ${currentBuild.currentResult}"
                }
            }
            post {
                    success {
                        echo "...QA Integration Testing Succeeded for ${env.BUILD_VERSION}: ${currentBuild.currentResult}"
                        publishHTML (target : [allowMissing: true,
                                                alwaysLinkToLastBuild: true,
                                                keepAll: true,
                                                reportDir: './newman',
                                                reportFiles: 'qa-integration-testing-report.html',
                                                reportName: 'QA Integration Test Reports',
                                                reportTitles: 'Test Report'])
                    } 
                    failure {
                        echo "...Promote API from Development Failed for ${env.BUILD_VERSION}: ${currentBuild.currentResult}"
                    }
                }  
        }
        
    
      stage('JMeter - Load Test') {
            steps {
                script {
                    def postman_envs = readJSON file: 'promote-api-output.json'
                    def client_id = postman_envs.environment.values.findIndexOf{ it.key == "client_id" }
                    def client_secret = postman_envs.environment.values.findIndexOf{ it.key == "client_secret" }
               
                    // sh 'mvn jmeter:configure jmeter:gui'   // get jmeter ui
                    sh """ mvn clean verify -DskipMunitTests \
                            -Dthreads=100 \
                            -Dduration=30 \
                            -Dapi_client_id=${postman_envs.environment.values[client_id].value} \
                            -Dapi_client_secret=${postman_envs.environment.values[client_secret].value} """
            
                }
            
            }
                  	post{
      		always{
	      			perfReport 'target/jmeter/results/*.csv'

                    publishHTML (target : [allowMissing: true,
                                            alwaysLinkToLastBuild: true,
                                            keepAll: true,
                                            reportDir: 'target/jmeter/reports/test1',
                                            reportFiles: 'index.html',
                                            reportName: 'JMeter Performance Reports',
                                            reportTitles: 'Performance Report'])
	    	 	}
      		
      		}
      	}    */

        stage('BDD') {
            steps {
                sh """ cd bdd && bat-cli """
            }
        }

    }
}
