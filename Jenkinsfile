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
                // sh 'mvn clean'
                // sh 'echo " this is building" '

                // sh './mvnw'
              }
            }
            stage('Run SonarQube Analysis') {
              elcaSonarqube.analyzeWithMaven(
                [
                        'sonar.projectKey'  : 'prj_elcavn-tech-training:devops-dotnet5',
                ]
              )
            }
          }

          container('oc') {
            elcaOKDLib = elcaOKDLoader.load()
            openshift.withCluster() {
              openshift.withProject('prj-elcavn-training-devops-intern') {
                stage('Build Docker Image') {
                  fileOperations([
                    fileCopyOperation(includes: 'target/petclinic.war', targetLocation: './docker', flattenFiles: true)
                  ])
                  openshift.apply(
                    openshift.process(
                      readFile('./okd/build.yml')
                    )
                  )
                  elcaOKDLib.buildAndWaitForCompletion('petclinic', '--from-dir ./docker')
                }

                stage('Deploy pet clinic Web') {
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


          // container('oc') {
          //   elcaOKDLib = elcaOKDLoader.load()
          //   openshift.withCluster() {
          //     openshift.withProject('prj-elcavn-training-luol') {
          //       stage('Build Docker Image') {
          //         fileOperations([
          //           fileCopyOperation(includes: 'target/sonar-ldap-permissions-*.jar', targetLocation: './docker', flattenFiles: true)
          //         ])
          //         openshift.apply(
          //           openshift.process(
          //             readFile('./okd/build.yml')
          //           )
          //         )
          //         elcaOKDLib.buildAndWaitForCompletion('sonarqube', '--from-dir ./docker')
          //       }

          //       stage('Deploy SonarQube') {
          //         openshift.apply(
          //           openshift.process(
          //             readFile('./okd/deploy.yml'),
          //             '-p', "CONTAINER_ID=${new Date().format("yyyyMMddHHmmssS", TimeZone.getTimeZone('UTC'))}",
          //           )
          //         )
          //         elcaOKDLib.waitForDeploymentCompletion('sonarqube')
          //       }
          //     }
          //   }
          // }

          // container('maven') {
          //   stage('Run analysis on newly deployed SonarQube') {
          //     withMaven() {
          //       sh "mvn sonar:sonar -Dsonar.host.url=http://sonarqube-prj-elcavn-training-luol.apps.okd.svc.elca.ch -Dsonar.login=admin -Dsonar.password=admin"
          //     }
          //   }
          // }
        }
      }
    }
  }
}
