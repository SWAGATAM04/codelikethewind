#! /usr/bin/env groovy

pipeline {
   environment {
    KUBERNETES_SERVICE_HOST = "api.demo1.xss4.p1.openshiftapps.com"
    KUBERNETES_SERVICE_PORT_HTTPS = "443"
   
    
  }
  agent any
    tools {
        maven "MAVEN"
        jdk "JDK"
    }
    stages {
        stage('Initialize'){
            steps{
                echo "PATH = ${M2_HOME}/bin:${PATH}"
                echo "M2_HOME = /opt/apache-maven-3.8.2"
                
            }
        }


    stage('Build') {
      steps {
        echo 'Building..'
        sh 'mvn clean package'
         sh 'jfrog c add --artifactory-url="https://swagatamjfrog.jfrog.io/artifactory/" --user="demo" --password="AKCp8mYy3yf14htMsfrogKCdsV9F16Kb7BuLMYSvoBpZPJcR6hWVwjgm1E69Wmb8FKuKQxATP"  --interactive="false"'
          sh 'jfrog rt u "target/simple-servlet-0.0.1-SNAPSHOT.war" "test/simple-servlet-0.0.1-SNAPSHOT-$BUILD_NUMBER.war" --recursive=false'
        
        // Add steps here
      }
    }
    stage('Create Container Image') {
      steps {
        echo 'Create Container Image..'
        
        script {

          // Add steps here
          openshift.withCluster() { 
            openshift.withCredentials('openshiftcred') {
                  openshift.withProject("swagatam-kundu-dev") {
  
               def buildConfigExists = openshift.selector("bc", "codelikethewind").exists() 
    
               if(!buildConfigExists){ 
                   openshift.newBuild("--name=codelikethewind", "--docker-image=registry.redhat.io/jboss-eap-7/eap74-openjdk8-openshift-rhel7", "--binary") 
               } 
    
                  openshift.selector("bc", "codelikethewind").startBuild("--from-file=target/simple-servlet-0.0.1-SNAPSHOT.war", "--follow") 
                  } 
            }
          }
        }
      }
    }
    stage('Deploy') {
      steps {
        echo 'Deploying....'
        script {

         openshift.withCluster() { 
          openshift.withCredentials('openshiftcred') {
            openshift.withProject("swagatam-kundu-dev") { 
                 def deployment = openshift.selector("dc", "codelikethewind") 
    
                  if(!deployment.exists()){ 
                    openshift.newApp('codelikethewind', "--as-deployment-config").narrow('svc').expose() 
                } 
    
                 timeout(5) { 
                    openshift.selector("dc", "codelikethewind").related('pods').untilEach(1) { 
                     return (it.object().status.phase == "Running") 
             } 
           } 
         } 
       }
     }
        }
      }
    }
  }
}
