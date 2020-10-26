pipeline {
 agent any

 options {
  skipDefaultCheckout()
 }
 stages {
  stage('SCM') {
   steps {
    checkout scm
   }
  }
  stage('Build') {
   parallel {
    stage('Compile') {
     agent {
      docker {
       image 'maven:3.6.0-jdk-8-alpine'
       args '-v /root/.m2/repository:/root/.m2/repository'
       reuseNode true
      }
     }
     steps {
      sh ' mvn clean compile'
     }
    }
    stage('CheckStyle') {
     agent {
      docker {
       image 'maven:3.6.0-jdk-8-alpine'
       args '-v /root/.m2/repository:/root/.m2/repository'
       reuseNode true
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
   steps {
    sh 'mvn test'
   }
   post {
    always {
     junit 'target/surefire-reports/**/*.xml'
    }
   }
  }
  stage('Integration Tests') {
  
   agent {
    docker {
     image 'maven:3.6.0-jdk-8-alpine'
     args '-v /root/.m2/repository:/root/.m2/repository'
     reuseNode true
    }
   }
   steps {
    sh 'mvn verify -Dsurefire.skip=true'
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
  }
  stage('Code Quality Analysis') {
   parallel {
    stage('PMD') {
     agent {
      docker {
       image 'maven:3.6.0-jdk-8-alpine'
       args '-v /root/.m2/repository:/root/.m2/repository'
       reuseNode true
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
     steps {
      sh ' mvn findbugs:findbugs'
      findbugs pattern: '**/target/findbugsXml.xml'
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
     steps {
      sh ' mvn javadoc:javadoc'
      step([$class: 'JavadocArchiver', javadocDir: './target/site/apidocs', keepAll: 'true'])
     }
    }
    stage('SonarQube') {
     agent {
      docker {
       image 'maven:3.6.0-jdk-8-alpine'
       args "-v /root/.m2/repository:/root/.m2/repository"
       reuseNode true
      }
     }
     steps {
      sh " mvn sonar:sonar -Dsonar.host.url=http://10.66.112.219:9000"
     }
    }
   }
   post {
    always {
     // using warning next gen plugin
     recordIssues aggregatingResults: true, tools: [javaDoc(), checkStyle(pattern: '**/target/checkstyle-result.xml'), findBugs(pattern: '**/target/findbugsXml.xml', useRankAsPriority: true), pmdParser(pattern: '**/target/pmd.xml')]
    }
   }
  }
 }
}
