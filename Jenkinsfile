pipeline {
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }

    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
           }
        stage('Checkout from Git'){
            steps{
                git branch: 'main', url: 'https://github.com/HIMA10SHREE/JENKINS-CICD-PIPELINE-AND-DEPLOY-TO-KUBERNETES-TAKING-SECURITY-PRACTICES.git'
            }
        }
        
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
                    -Dsonar.projectKey=Netflix '''
                }
            }
        }
        
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        
        
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){   
                       sh "docker build --build-arg TMDB_V3_API_KEY=74fdb74ee243f5e581b2a2adc1d35716 -t netflix ."
                       sh "docker tag netflix hima10shree/netflix:latest "
                       sh "docker push hima10shree/netflix:latest "
                    }
                }
            }
        }
        
        stage("TRIVY"){
            steps{
                sh "trivy image hima10shree/netflix:latest > trivyimage.txt" 
            }
        }
        // stage('Deploy to container'){
        //     steps{
        //         sh 'docker run -d -p 8081:80 hima10shree/netflix:latest'
        //     }
        // }
        
        stage('Deploy to Kubernetes Cluster'){
            steps{
                script{
                    dir('Kubernetes'){
                            withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'kubernetes', namespace: '', restrictKubeConfigAccess: false, serverUrl: ''){
                            sh 'kubectl delete --all pods'
                            sh 'kubectl apply -f deployment.yml --validate=false'
                            sh 'kubectl apply -f service.yml --validate=false'
                        }
                    }
                    
                }
            }
        }
    } 
    post {
        always {
                emailext attachLog: true,
                    subject: "'${currentBuild.result}'",
                    body: "Project: ${env.JOB_NAME}<br/>" +
                        "Build Number: ${env.BUILD_NUMBER}<br/>" +
                        "URL: ${env.BUILD_URL}<br/>",
                    to: 'hazarikahimashree94@gmail.com',                              
                    attachmentsPattern: 'trivyfs.txt,trivyimage.txt'
                }
            }
            
}
