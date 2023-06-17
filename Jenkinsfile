#!/usr/bin/env groovy
// By Erivando Sena

pipeline {
    agent {
        kubernetes {
            defaultContainer 'maven'
            yaml """
                apiVersion: v1
                kind: Pod
                metadata:
                  name: jenkins-agent-inbound
                  namespace: jenkins
                  labels:
                    app: jenkins-agent-inbound
                  annotations:
                    cluster-autoscaler.kubernetes.io/safe-to-evict: "false"
                    datree.skip/CONTAINERS_MISSING_READINESSPROBE_KEY: readiness probe não é relevante para contêineres de shell CI/CD CLI
                    datree.skip/CONTAINERS_MISSING_LIVENESSPROBE_KEY: liveness probe não é relevante para contêineres de shell CI/CD CLI
                spec:
                  serviceAccountName: jenkins-admin
                  priorityClassName: high-priority-apps
                  affinity:
                    podAntiAffinity:
                      preferredDuringSchedulingIgnoredDuringExecution:
                        - weight: 100
                          podAffinityTerm:
                            topologyKey: topology.kubernetes.io/zone
                            labelSelector:
                              matchExpressions:
                                - key: app
                                  operator: In
                                  values:
                                    - jenkins-agent
                        - weight: 100
                          podAffinityTerm:
                            topologyKey: kubernetes.io/hostname
                            labelSelector:
                              matchExpressions:
                                - key: app
                                  operator: In
                                  values:
                                    - jenkins-agent
                  securityContext:
                    runAsUser: 0
                  containers:  
                    # ==================================================
                    # Container Maven (Default)
                    # ==================================================
                    - name: maven
                      image: jenkins/inbound-agent:latest
                      command: ["tail", "-f", "/dev/null"]
                      imagePullPolicy: IfNotPresent
                      resources:
                        requests:
                          cpu: "250m"
                          memory: "512Mi"
                        limits:
                          cpu: "4000m"
                          memory: "4Gi"
                    # ==================================================
                    # Container PHP
                    # ==================================================
                    - name: php8
                      image: php:8.1-apache-bullseye
                      command: ["bash", "-c", "apt-get update && apt-get install -y git && tail -f /dev/null"]
                      imagePullPolicy: IfNotPresent
                      resources:
                        requests:
                          cpu: "250m"
                          memory: "512Mi"
                        limits:
                          cpu: "4000m"
                          memory: "8Gi"
                    # ==================================================
                    # Conatiner Docker
                    # ==================================================
                    - name: docker
                      image: docker:20.10.23-cli
                      command: ["tail", "-f", "/dev/null"]
                      imagePullPolicy: IfNotPresent
                      resources:
                        requests:
                          cpu: "250m"
                          memory: "512Mi"
                        limits:
                          cpu: "4000m"
                          memory: "2Gi"
                      volumeMounts:
                        - name: docker
                          mountPath: /var/run/docker.sock
                  volumes:
                    - name: docker
                      hostPath:
                  imagePullSecrets:
                  - name: harbor-regcred-k8sc3
                """
        }
    }
    
    options {
        timestamps()
        timeout(time: 24, unit: 'HOURS')
        parallelsAlwaysFailFast()
        rateLimitBuilds(throttle: [count: 3, durationName: 'minute', userBoost: false])
        buildDiscarder(logRotator(numToKeepStr: '20'))
        disableConcurrentBuilds()
    }
    
    triggers {
        cron('0 0 * * 1') 
    }
    
    environment {
        APP_NAME = "app"
        GIT_TAG = "v.1.0"
        DOCKER_IMAGE = "erivando/${APP_NAME}:${GIT_TAG}"
    }
    
    stages {
        stage('Iniciando CI/CD') {
            steps {
                milestone(ordinal: null, label: "Milestone: Setup")
    
                script {
                    workspace = "$env.WORKSPACE"
                }
    
                // executar alguns comandos shell para configurar detalhes
                sh '''
                    # Instala pacotes
                    apt-get update && apt-get upgrade -y 
                    apt-get install -y sudo curl unzip
                '''
            }
        }
        
        stage('Checkout') {
            steps {
                sh '''
                apt update && apt install -y git
                
                git config --global user.name "Up DevOps"
                git config --global user.email "seniorfullstackdevops@gmail.com"
                '''
                checkout([$class: 'GitSCM',
                    extensions: [[$class: 'CleanBeforeCheckout']],
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/erivandosena/pipeline-cicd-template.git',
                        credentialsId: 'git-user-credential'
                    ]]
                ])
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                container('maven') {
                    script {
                        sh 'apontar para seu scanner de teste de qualidade e devsecops'
                    }
                } 
            }
        }
        
        stage('Testes') {
            steps {
                sh 'echo "apontar para suas classes de testes"'
            }
        }
        
        stage('Build') {
            steps {
                container('docker') {
                    sh "echo 'build da application'"
                }
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    sh ''' #!/bin/bash 
                        echo "deploy incluindo validação manual se for o caso"
                    '''
                }
            }
        }
    }
    
    post{
        always {
            echo 'Processo de pipeline finalizado.'
        }
        success {
            echo 'Processo de pipeline BEM-SUCEDIDO.'
        }
        failure {
            echo 'Processo de pipeline executado com falha.'
        }
        unstable {
            echo 'Processo de pipeline possui estabilidade.'
        }
        changed {
            echo 'SUCESSO no processo de pipeline anteriormente falhado.'
        }
    }  
}
