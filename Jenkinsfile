pipeline {
    agent any

    tools {
        nodejs "node-24-8-0"
    }

    environment {
         MONGO_URI = "mongodb+srv://cluster0.xgt6cyx.mongodb.net/planets"
         SONAR_SCANNER_HOME=  tool 'sonarqube-scanner'
         GITEA_TOKEN=credentials('gitt')
           }


    stages {
        stage('Install Dependencies') {
            options
            {
                timestamps()
            }
            steps {
                sh 'npm install --no-audit'
            }
        }

        stage('Security Scans') {
            parallel {
                stage('NPM Audit Critical') {
                    steps {
                        sh 'npm audit --audit-level=critical || true'
                    }
                }

                stage('OWASP Dependency Check') {
                    steps {
                        dependencyCheck additionalArguments: '''
                            --scan './' 
                            --out './' 
                            --format 'ALL' 
                            --prettyPrint
                        ''',
                        odcInstallation: 'OWASP-DepCheck-10'

                        dependencyCheckPublisher(
                            failedTotalCritical: 4,
                            pattern: 'dependency-check-report.xml',
                            stopBuild: true
                        )

                        
                    }
                }
                
            }
        }
        stage('unit testing') {
                    steps
            0        { 
                        withCredentials([usernamePassword(credentialsId: 'monog-db-cred', passwordVariable: 'MONGO_PASSWORD', usernameVariable: 'MONGO_USERNAME')]) {
                         sh 'npm test || true'

                         } 
                        // junit allowEmptyResults: true, stdioRetention: '', testResults: 'test-results.xml'

                    }
                }
        stage('Code Coverage') {
                    steps
                    { 
                        withCredentials([usernamePassword(credentialsId: 'monog-db-cred', passwordVariable: 'MONGO_PASSWORD', usernameVariable: 'MONGO_USERNAME')]) {
                           catchError(buildResult: 'SUCCESS', message: 'oops! seems to be an error  but no problemm will be fixed', stageResult: 'UNSTABLE') {

                            sh 'npm run coverage'

}
                           
                         } 

                       

                        
                    }
                }     
               

        stage('SAST - SonarQube') {
            steps {
                timeout(time: 60, unit: 'SECONDS') {
                    withSonarQubeEnv('sonar-qube-server') {
                        sh '''
                            $SONAR_SCANNER_HOME/bin/sonar-scanner \
                            -Dsonar.projectKey=solar-system-project \
                            -Dsonar.sources=app.js \
                            -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info
                        '''

                        
                    }
                    waitForQualityGate abortPipeline: true
                }
            }
        } 

        stage('Build Docker Image')
        {


            steps{
                sh 'docker build -t sudhanshgoyal/solarsystem:$GIT_COMMIT .' 
            }
        }

        stage('Trivy Vulnerability Scanner') {
    steps {
        sh '''
            trivy image sudhanshgoyal/solarsystem:$GIT_COMMIT \
                --severity LOW,MEDIUM,HIGH \
                --exit-code 0 \
                --quiet \
                --format json -o trivy-image-MEDIUM-results.json

            trivy image sudhanshgoyal/solarsystem:$GIT_COMMIT \
                --severity CRITICAL \
                --exit-code 0 \
                --quiet \
                --format json -o trivy-image-CRITICAL-results.json
        ''' 

    }
    post {
    always {
        sh '''
             # Convert MEDIUM results to HTML
            trivy convert \
                --format template \
                --template "@/usr/local/share/trivy/templates/html.tpl" \
                --output trivy-image-MEDIUM-results.html \
                trivy-image-MEDIUM-results.json

            # Convert CRITICAL results to HTML
            trivy convert \
                --format template \
                --template "@/usr/local/share/trivy/templates/html.tpl" \
                --output trivy-image-CRITICAL-results.html \
                trivy-image-CRITICAL-results.json

            # Convert MEDIUM results to JUnit XML
            trivy convert \
                --format template \
                --template "@/usr/local/share/trivy/templates/junit.tpl" \
                --output trivy-image-MEDIUM-results.xml \
                trivy-image-MEDIUM-results.json

            # Convert CRITICAL results to JUnit XML
            trivy convert \
                --format template \
                --template "@/usr/local/share/trivy/templates/junit.tpl" \
                --output trivy-image-CRITICAL-results.xml \
                trivy-image-CRITICAL-results.json
        '''

        
    }
}

}
       stage('Push Docker Image')
        {
            steps{
                
                withDockerRegistry(credentialsId: 'docker111', url: '') {
 

                sh 'docker push sudhanshgoyal/solarsystem:$GIT_COMMIT' 
            }}
        }

        stage('Deploy aws instance')
        {
            steps{
                
              sshagent(['ssh-private-credentials']) {
   
            }
            }}

                stage('Integrate Testing -AWS EC2') {
            steps {
             withCredentials([usernamePassword(credentialsId: 'monog-db-cred', passwordVariable: 'MONGO_PASSWORD', usernameVariable: 'MONGO_USERNAME')]) {
                sshagent(['ssh-private-credentials']) {
                  sh """
                    ssh -o StrictHostKeyChecking=no ubuntu@52.90.171.26 \\
                    "sudo docker stop solar-system || true && \\
                     sudo docker rm solar-system || true && \\
                     sudo docker run --name solar-system \\
                        -e MONGO_URI='mongodb+srv://$MONGO_USERNAME:$MONGO_PASSWORD@cluster0.xgt6cyx.mongodb.net/planets' \\
                        -p 3000:3000 -d sudhanshgoyal/solarsystem:$GIT_COMMIT"
                  """
            }
        }
    }
}

             stage('K8S Update Image Tag') {
            when {
                branch 'PR*'
            }
            steps {
                sh 'git clone -b main http://54.235.21.182:3000/sudhansh/solar-system-gitops-argocd'
                dir("solar-system-gitops-argocd/kubernetes") {
                    sh '''
                        git checkout main
                        git checkout -b feature-$BUILD_ID
                        sed -i "s|sudhanshgoyal/solarsystem:.*|sudhanshgoyal/solarsystem:$GIT_COMMIT|" deployment.yml
                        git config --global user.email "jenkins@ci.local"
                        git remote set-url origin http://$GITEA_TOKEN@54.235.21.182:3000/sudhansh/solar-system-gitops-argocd
                        git add .
                        git commit -am "Updated docker image"
                        git push -u origin feature-$BUILD_ID
                    '''
                }
            }
        }
         
         
         stage('K8S - Raise PR') {
    when {
        branch 'PR*'
    }
    steps {
        sh """
            curl -X POST \
              'http://54.235.21.182:3000/api/v1/repos/sudhansh/solar-system-gitops-argocd/pulls' \
              -H 'accept: application/json' \
              -H 'Authorization: token $GITEA_TOKEN' \
              -H 'Content-Type: application/json' \
              -d '{
                "assignee": "sudhansh",
                "assignees": ["sudhansh"],
                "base": "main",
                "body": "Updated docker image in deployment manifest",
                "head": "feature-$BUILD_ID",
                "title": "Updated Docker Image"
              }'
        """
    }
}

stage('App Deployed?') {
    when {
        branch 'PR*'
    }
    steps {
        timeout(time: 1, unit: 'DAYS') {
            input message: 'Is the PR Merged and ArgoCD Synced?', 
                  ok: 'YES! PR is Merged and ArgoCD Applied'
        }
    }
}

// stage('DAST - OWASP ZAP') {
//     when {
//         branch 'PR*'
//     }
//     steps {
//         sh '''
           
//             chmod 777 $(pwd)
//              cat > zap_ignore_rules <<EOF
// 100001\tIGNORE\thttp://54.235.21.182:30000
// 10020\tIGNORE\thttp://54.235.21.182:30000
// 10021\tIGNORE\thttp://54.235.21.182:30000
// 10037\tIGNORE\thttp://54.235.21.182:30000
// 10038\tIGNORE\thttp://54.235.21.182:30000
// 10063\tIGNORE\thttp://54.235.21.182:30000
// 10098\tIGNORE\thttp://54.235.21.182:30000
// 90003\tIGNORE\thttp://54.235.21.182:30000
// 90004\tIGNORE\thttp://54.235.21.182:30000
// EOF
//             docker run -v $(pwd):/zap/wrk/:rw \
//               ghcr.io/zaproxy/zaproxy zap-api-scan.py \
//               -t http://54.235.21.182:30000/api-docs/ \
//               -f openapi \
//               -r zap_report.html \
//               -w zap_report.md \
//               -J zap_json_report.json \
//               -x zap_xml_report.xml \
//               -c zap_ignore_rules
//         '''
//     }
// }

stage('Upload -- AWS S3') {
    when {
        branch 'PR*'
    }
    steps {
    withAWS(credentials: 'EC2-S3-LAMBDA', region: 'us-east-1') 
 {
        sh '''
            ls -ltr
            mkdir reports-$BUILD_ID
            cp -rf coverage/ reports-$BUILD_ID/
            cp dependency*.* trivy*.* zap*.* reports-$BUILD_ID/
            ls -ltr reports-$BUILD_ID/
        '''
        s3Upload(
            file: "reports-$BUILD_ID",
            bucket: 'solar-system-jenkins-reports-bucket-1111111111111111111111',
            path: "jenkins-$BUILD_ID/",
               // <-- Add your AWS credentials ID here
        )
    }
}

}


        }


    post
    {
        always
        {
            script {
            if (fileExists('solar-system-gitops-argocd')) {
                sh 'rm -rf solar-system-gitops-argocd'
            }
            }
        
            publishHTML([
                            allowMissing: true,
                            alwaysLinkToLastBuild: true,
                            keepAll: true,
                            reportDir: './',
                            reportFiles: 'dependency-check-jenkins.html',
                            reportName: 'Dependency Check HTML Report',
                            useWrapperFileDirectly: true
                        ])

                        junit(
                            allowEmptyResults: true,
                            testResults: 'dependency-check-junit.xml'
                        )

            junit(
                            allowEmptyResults: true,
                            testResults: 'test-results.xml'
                        
                        )
             publishHTML([
                            allowMissing: true,
                            alwaysLinkToLastBuild: true,
                            keepAll: true,
                            reportDir: './',
                            reportFiles: 'zap_report.html',
                            reportName: 'DAST - OWASP ZAP REPORT',
                            useWrapperFileDirectly: true
                        ])             

             publishHTML([
                            allowMissing: true,
                            alwaysLinkToLastBuild: true,
                            keepAll: true,
                            reportDir: 'coverage/lcov-report',
                            reportFiles: 'index.html',
                            reportName: 'Code Coverage HTML Report',
                            useWrapperFileDirectly: true
                        ])
            // Publish HTML reports in Jenkins
             publishHTML([
            allowMissing: true,
            alwaysLinkToLastBuild: true,
            keepAll: true,
            reportDir: '.',
            reportFiles: 'trivy-image-MEDIUM-results.html',
            reportName: 'Trivy Medium Vulnerabilities'
        ])

        publishHTML([
            allowMissing: true,
            alwaysLinkToLastBuild: true,
            keepAll: true,
            reportDir: '.',
            reportFiles: 'trivy-image-CRITICAL-results.html',
            reportName: 'Trivy Critical Vulnerabilities'
        ])
            junit(
                            allowEmptyResults: true,
                            testResults: 'trivy-image-MEDIUM-results.xml'
                        ) 
            junit(
                            allowEmptyResults: true,
                            testResults: 'trivy-image-CRITICAL-results.xml'
                        )             
}
    }
}
