# DevOps - Jenkins Scripts

- [DevOps - Jenkins Scripts](#devops---jenkins-scripts)
  - [Email Notification Script](#email-notification-script)
    - [Pipeline code](#pipeline-code)
    - [Explanation](#explanation)
    - [Plugin Required](#plugin-required)

## Email Notification Script
- Contains a jenkins script to send email notification on build failure
- A modular pipeline should that can be embeded in any other pipe line

### Pipeline code
```groovy
node {
    // config
    def to = emailextrecipients([
        [$class: 'CulpritsRecipientProvider'],
        [$class: 'DevelopersRecipientProvider'],
        [$class: 'RequesterRecipientProvider']
    ])
    try {
        stage('build') {
            println('so far so good...')
        }
        stage('test') {
            println('A test has failed!')
            sh 'exit 1'
        }
    } catch (e) {
        currentBuild.result = 'FAILURE'

        def subject = "${env.JOB_NAME} - Build #${env.BUILD_NUMBER} ${currentBuild.result}"
        def content = '${JELLY_SCRIPT,template="html"}'

        if (to != null && !to.isEmpty()) {
            emailext(body: content, mimeType: 'text/html',
            replyTo: '$DEFAULT_REPLYTO', subject: subject,
            to: to, attachLog: true )
        }
        throw e
    }
}

```

### Explanation
- `$class` represents the various types of recepeints
  - `DevelopersRecipientProvider` developer whoe code failed
  - `CulpritsRecipientProvider` Manager
  - `RequesterRecipientProvider` Person who mnually executed the build
- The mail be wont be sent if they are not specified in ettings
- Build is purposely made to fail in try block
- Catch block is used to send the email
- Subjects and Content is defined in catch block
- `${JELLY_SCRIPT,template="html"}` provides a fancy UI to show logs (provided by email extension plugin)
- If atleast one of the recepient is not null then the email will be sent

### Plugin Required
- `Email Extension Plugin`