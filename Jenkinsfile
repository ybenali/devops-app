pipeline {
  agent any
  stages {
    stage('SCM') {
      steps {
        checkout scm
      }
    }

    stage('Build') {
      agent {
        docker {
          image 'maven:3.6.0-jdk-8-alpine'
          args '-p 3011:3011'
          reuseNode true
        }

      }
      steps {
        sh ''' mvn clean compile
#mvn clean
#mvn spring-boot:run'''
      }
    }

    stage('Code Quality Analysis') {
      parallel {
        stage('CheckStyle') {
          agent {
            docker {
              image 'maven:3.6.0-jdk-8-alpine'
              args '-v /root/.m2/repository:/root/.m2/repository'
              reuseNode true
            }

          }
          when {
            anyOf {
              branch 'develop'
            }

          }
          steps {
            sh ' mvn checkstyle:checkstyle'
            step([$class: 'CheckStylePublisher',
                                                                                                                                                                   //canRunOnFailed: true,
                                                                                                                                                                   defaultEncoding: '',
                                                                                                                                                                   healthy: '100',
                                                                                                                                                                   pattern: '**/target/checkstyle-result.xml',
                                                                                                                                                                   unHealthy: '90',
                                                                                                                                                                   //useStableBuildAsReference: true
                                                                                                                                                                  ])
          }
        }

        stage('PMD') {
          agent {
            docker {
              image 'maven:3.6.0-jdk-8-alpine'
              args '-v /root/.m2/repository:/root/.m2/repository'
              reuseNode true
            }

          }
          when {
            anyOf {
              branch 'develop'
            }

          }
          steps {
            sh ' mvn pmd:pmd'
            step([$class: 'PmdPublisher', pattern: '**/target/pmd.xml'])
          }
        }

        stage('Findbugs') {
          agent {
            docker {
              image 'maven:3.6.0-jdk-8-alpine'
              args '-v /root/.m2/repository:/root/.m2/repository'
              reuseNode true
            }

          }
          when {
            anyOf {
              branch 'develop'
            }

          }
          steps {
            sh ' mvn findbugs:findbugs'
            findbugs(pattern: '**/target/findbugsXml.xml')
          }
        }

        stage('JavaDoc') {
          agent {
            docker {
              image 'maven:3.6.0-jdk-8-alpine'
              args '-v /root/.m2/repository:/root/.m2/repository'
              reuseNode true
            }

          }
          when {
            anyOf {
              branch 'develop'
            }

          }
          steps {
            sh ' mvn javadoc:javadoc'
            step([$class: 'JavadocArchiver', javadocDir: './target/site/apidocs', keepAll: 'true'])
          }
        }

        stage('SonarQube') {
          agent {
            docker {
              image 'maven:3.6.0-jdk-8-alpine'
              args '-v /root/.m2/repository:/root/.m2/repository'
              reuseNode true
            }

          }
          when {
            anyOf {
              branch 'develop'
            }

          }
          steps {
            sh ' mvn sonar:sonar -Dsonar.host.url=http://10.66.12.219:9000'
          }
        }

      }
    }

    stage('Unit Tests') {
      agent {
        docker {
          image 'maven:3.6.0-jdk-8-alpine'
          args '-v /root/.m2/repository:/root/.m2/repository'
          reuseNode true
        }

      }
      when {
        anyOf {
          branch 'develop'
        }

      }
      post {
        always {
          junit 'target/surefire-reports/**/*.xml'
        }

      }
      steps {
        sh 'mvn test'
      }
    }

    stage('packaging (Docker)') {
      agent {
        docker {
          image 'maven:3.6.0-jdk-8-alpine'
          args '-v /root/.m2/repository:/root/.m2/repository'
          reuseNode true
        }

      }
      when {
        anyOf {
          branch 'develop'
        }

      }
      post {
        always {
          junit 'target/failsafe-reports/**/*.xml'
        }

        success {
          stash(name: 'artifact', includes: 'target/*.war')
          stash(name: 'pom', includes: 'pom.xml')
          archiveArtifacts 'target/*.war'
        }

      }
      steps {
        sh 'mvn verify -Dsurefire.skip=true'
      }
    }

    stage('Push to git') {
      when {
        anyOf {
          branch 'develop'
        }

      }
      steps {
        sh '''
            pwd
            #git remote add origin https://github.com/ybenali/devops-app.git
            git branch -M main
cd  /var/jenkins_home/workspace/devops-app_main/target
            git add -f demo-0.0.1-SNAPSHOT.war
            git commit -m "Push war file"
#           git push -u \'youssef.benali@altersis.com\' -p \'usefBA29!\' test  main


git push https://ybenali:Welcomecpt2020@github.com/ybenali/devops-app.git
            '''
      }
    }

    stage('Deployment (Docker)') {
      parallel {
        stage('UAT') {
          agent {
            dockerfile {
              filename 'app/Dockerfile'
              reuseNode true
            }

          }
          steps {
            sh '''cd /usr/local/tomcat/bin

./startup.sh'''
          }
        }

        stage('Staging') {
          steps {
            sh 'echo \'staging\''
          }
        }

      }
    }

    stage('Tools Deployment [Docker]') {
      steps {
        sh '''docker run -d --restart=unless-stopped --privileged=true --pid=host --net=host --ipc=host -v /:/mnt/root -e ONEAGENT_INSTALLER_SCRIPT_URL=https://yyl00213.live.dynatrace.com/api/v1/deployment/installer/agent/unix/default/latest?arch=x86&flavor=default   -e ONEAGENT_INSTALLER_DOWNLOAD_TOKEN=rNVzbAFwT6SnOvx2ehmVR dynatrace/oneagent --set-app-log-content-access=true
'''
      }
    }

  }
}