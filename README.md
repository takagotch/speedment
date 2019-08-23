### speedment
---
https://github.com/speedment/speedment

```
```

```sql
SELECT 
  `film_id`,`title`,`description`,`release_year`,
  `language_id`,`original_language_id`,`rental_duration`,`rental_rate`
  `length`,`replacement_cost`,`rating`,`special_features`,
  `last_update`
FROM 
  `akila`.`film`
WHERE
  (`length` > 120)
```

```jenkins
pipeline {
  agent any
  
  stages {
    stage('Build') {
      steps {
        mvn 'clean install -DskipTests'
      }
    }
    
    stage('Unit Test') {
      steps {
        mvn 'test'
      }
    }
    
    //stage('Integration Test') {
    //  steps {
    //    //mvn '-Prelease test'
    //    mvn 'test'
    //}
    //}
  }
  
  post {
    always {
      junit allowEmptyResults: true,
          testResults: '**/target/surefile-reports/TEST-*.xml, **/target/failsafe-reports/*.xml'
      mailIfStatusChanged env.EMAIL_RECIPIENTS
    }
    success {
      slackSend (color: "good", message: "Build unstable\nBuild: ${env.JOB_NAME} #${env.BUILD_NUMBER}\nURL: ${env.BUILD_URL}")
    }
    failure {
      slackSend (color: "danger", message: "Build failed\nBuild: ${env.JOB_NAME} #${env.BUILD_NUMBER}\nURL: ${env.BUILD_URL}")
    }
  }
}

def mailIfStatusChanged(String recipients) {
  if (currentBuild.currentResult == 'SUCCESS') {
    currentBuild.result = 'SUCCESS'
  }
  step([$class: 'Mailer', recipients: recipients])
}

def mvn(def args) {
  def mvnHome = tool 'M3'
  def javaHome = tool 'JDK8'
  
  withEnv(["JAVA_HOME=${javaHome}", "PATH+MAVEN=${mvnHome}/bin:${env.JAVA_HOME/bin}"]) {
    sh "$(mvnHome)/bin/mvn ${args} --batch-mode -V -U -e Dsurefire.useFile=false"
  }
}
```


