pipeline {
    agent any
    // tools {
    //     maven 'maven3'
    // }
    environment {
        SCANNER_HOME= tool 'sonar-scanner-7.3.0'
    }
    stages {
        
        stage("Build Code") {
            tools {
                maven 'maven-3.9.11'
            }
            steps {
                sh "mvn clean install -DskipTests"
            }
        }

        stage('Trivy FS scan') {
        steps {
        sh '''
            docker run --rm \
            -v $(pwd):/project \
            aquasec/trivy:latest \
            fs --format table -o /project/fs.html /project
        '''
            }
        }
        // stage('Trivy FS scan') {
        //     steps {
        //         sh 'trivy fs --format table -o fs.html .'
        //     }
        // }     
        stage('Sonar Analysis') {
            steps {
                withSonarQubeEnv('sonar-local') {
                    sh '''  $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=blogging -Dsonar.projectKey=blogging \
                    -Dsonar.java.binaries=target '''
                }
            }
        }
        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage("Upload Artifacts") {
            steps {
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: 'nexus:8081',
                    groupId: 'com.master',
                    version: '0.0.1-SNAPSHOT',   // must match POM
                    repository: 'maven-snapshots',  // snapshot repo
                    credentialsId: 'nexus-jenkins-creds',
                    artifacts: [
                     [
                        artifactId: 'todo-app',     // must match POM
                        classifier: '',
                        file: 'target/todo-app-1.0-SNAPSHOT.jar',
                        type: 'jar'
                    ]
                    ]   
                )
            }
        }
        // stage('Publish Artifacts') {
        //     steps {
        //         withMaven(globalMavenSettingsConfig: 'settings-maven', jdk: 'jdk17', maven: 'maven3', traceability: true) {
        //                 sh 'mvn deploy'
        //         }
        //     }
        // }
        stage('Docker build and Tag') {
            steps {
                script{
                withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                        sh 'docker build -t ash425/taskmaster:latest .'
                    }
                }
            }
        }
       stage('Trivy Image Scan') {
        steps {
        sh '''
            docker run --rm \
              -v /var/run/docker.sock:/var/run/docker.sock \
              -v $WORKSPACE:/workspace \
              aquasec/trivy:latest \
              image --format table -o /workspace/image-scan-report.html ash425/taskmaster:latest
        '''
            }
       }
                // stage('Trivy Image scan') {
        //     steps {
        //         sh 'trivy image --format table -o image.html jay24666/taskmaster:latest'
        //     }
        // }
        stage('Push Docker Image') {
            steps {
                script{
                withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                        sh 'docker push ash425/taskmaster:latest'
                    }
                }
            }
        }
         stage ("Deploy to cluster dev-kt-k8s") {
            steps {
                withKubeConfig(credentialsId: 'minikube-kubeconfig') {
                    sh "kubectl apply -f deployment-service.yml --validate=false"
                }  
            }
         }
        // stage('K8s Deploy') {
        //     steps {
        //         withKubeConfig(caCertificate: '', clusterName: ' blog-cluster', contextName: '', credentialsId: 'k8s-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://F215F65BF29C7EB75F58C53DC3D1C08C.gr7.us-east-1.eks.amazonaws.com') {
        //                 sh 'kubectl apply -f deployment-service.yml'
        //                 sleep 35
        //             }
        //     }
        // }
        // stage('Verify K8s Deploy') {
        //     steps {
        //         withKubeConfig(caCertificate: '', clusterName: ' blog-cluster', contextName: '', credentialsId: 'k8s-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://F215F65BF29C7EB75F58C53DC3D1C08C.gr7.us-east-1.eks.amazonaws.com') {
        //                 sh 'kubectl get pods -n webapps'
        //                 sh 'kubectl get svc -n webapps'
        //             }
        //     }
        // }
    }
}

