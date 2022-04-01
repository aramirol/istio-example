// JENKINSFILE

// Defining Credentials

// Set working directory

// Init pipeline
pipeline{
    agent any

    options {
        ansiColor('xterm')
    }

    stages {
        stage("Getting Namespace Exist") {
            steps {
                script {
                    NAMESPACE_EXIST = sh (
                        script: "kubectl get namespaces --kubeconfig=/var/jenkins_home/.kube/kubeconfig/config | grep istio-example-bg | wc -l",
                        returnStdout: true
                    ).trim()
                    echo "Namespace Count: ${NAMESPACE_EXIST}"
                }
            }
        }

        stage("Getting Active Color") {
            steps {
                script {
                    if ("${NAMESPACE_EXIST}" == "1") {
                        ACTIVE_COLOR = sh (
                            script: "kubectl get virtualservice -n istio-example-bg myapp-virtualservice -o jsonpath='{.spec.http[1].route[0].destination.subset}' --kubeconfig=/var/jenkins_home/.kube/kubeconfig/config",
                            returnStdout: true
                        ).trim()
                    } else {
                        if ("${NAMESPACE_EXIST}" == "0") {
                            ACTIVE_COLOR = "white"
                        }
                    }
                    echo "Active Color: ${ACTIVE_COLOR}"
                }
            }
        }

        stage("Setting New Color") {
            steps {
                script {
                    if ("${ACTIVE_COLOR}" == 'blue') {
                        NEW_COLOR = "green"
                    } else {
                        if ("${ACTIVE_COLOR}" == 'green') {
                            NEW_COLOR = "blue"
                        } else {
                            if ("${ACTIVE_COLOR}" == 'white') {
                                NEW_COLOR = "white"
                            }
                        }
                    }
                    echo "New Color: ${NEW_COLOR}"
                }
            }
        }

        stage('Deploying App') {
            steps {
                script {
                    if ("${ACTIVE_COLOR}" == 'white') {
                        sh "kubectl apply -f kubernetes/all.yml --kubeconfig=/var/jenkins_home/.kube/kubeconfig/config"
                    } else {
                        sh "kubectl apply -f kubernetes/deployment_${NEW_COLOR}.yml --kubeconfig=/var/jenkins_home/.kube/kubeconfig/config"
                    }
                }
            }
        }

        stage('Testing App') {
            steps {
                script {
                    if ("${ACTIVE_COLOR}" == 'white') {
                        sh "curl -Is http://app.istio.demo.lab:32355 | head -n 1"
                    } else {
                        sh "curl -Is --cookie test=true http://app.istio.demo.lab:32355 | head -n 1"
                    }
                }
            }
        }

        stage('Switching Virtual Service') {
            steps {
               script {
                   if ("${ACTIVE_COLOR}" != 'white') {
                        sh "kubectl replace -f kubernetes/virtualservice_${NEW_COLOR}.yml -n istio-example-bg --kubeconfig=/var/jenkins_home/.kube/kubeconfig/config"
                   }
                } 
            }
        }
    }

    post {
        cleanup {
            cleanWs()
        }
    }
}