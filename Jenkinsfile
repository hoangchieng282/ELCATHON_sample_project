#!/usr/bin/env groovy

timestamps {
  elcaPodTemplates.base {
    elcaPodTemplates.maven([
    // Docker image tag. Default is 'eclipse-temurin'. See https://hub.docker.com/_/maven?tab=tags
    tag: '3.8-eclipse-temurin-8', // Picking up the latest release of Maven 3.8.x and latest release of Eclipse Temurin JDK 8
  ]) {
      elcaPodTemplates.oc {
        node(POD_LABEL) {
          elcaStage.gitCheckout()

          container('maven') {
            stage('Build') {
              withMaven() {
                sh 'mvn clean install'
              }
            }
            stage('Run SonarQube Analysis') {
              elcaSonarqube.analyzeWithMaven(
                // [
                //         'sonar.projectKey'  : 'prj_elcavn-tech-training:devops-dotnet6',
                // ]
                )
            }
          }

          container('oc') {
            elcaOKDLib = elcaOKDLoader.load()
            openshift.withCluster() {
              openshift.withProject('prj-elcavn-training-chih') {
                stage('Build Docker Image') {
                  fileOperations([
                    fileCopyOperation(includes: 'target/*.war', targetLocation: './docker', flattenFiles: true)
                  ])
                  openshift.apply(
                    openshift.process(
                      readFile('./okd/build.yml')
                    )
                  )
                  elcaOKDLib.buildAndWaitForCompletion('petclinic', '--from-dir ./docker')
                }

                stage('Deploy petclinic web') {
                  openshift.apply(
                    openshift.process(
                      readFile('./okd/deploy.yml'),
                      '-p', "CONTAINER_ID=${new Date().format("yyyyMMddHHmmssS", TimeZone.getTimeZone('UTC'))}",
                    )
                  )
                  elcaOKDLib.waitForDeploymentCompletion('petclinic')
                }
              }
            }
          }

          // container('maven') {
          //   stage('Run analysis on newly deployed SonarQube') {
          //     withMaven() {
          //       sh "mvn sonar:sonar -Dsonar.host.url=http://sonarqube-prj-elcavn-training-chih.apps.okd.svc.elca.ch -Dsonar.login=admin -Dsonar.password=admin"
          //     }
          //   }
          // }
        }
      }
    }
  }
}