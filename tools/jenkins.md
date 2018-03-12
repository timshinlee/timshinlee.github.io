> [Jenkins User Documentation](https://jenkins.io/doc/)

# Pipeline

Pipeline就是通过代码的方式把软件从版本控制传递到用户手上的工具。

Pipeline示例：

```java
// 文件名为Jenkinsfile
pipeline {
    agent { docker { image 'maven:3.3.3' } }
    stages {
        stage('build') {
            steps {
                sh 'mvn --version'
	    }
	}
    }
}
```

pipeline的语法当中，不同操作系统执行命令的关键字不同：

- linux和MacOS中，使用`sh`执行shell命令
- Windows中，使用`bat`执行batch命令

```java
Jenkinsfile (Declarative Pipeline)
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'echo "Hello World"'
		sh ```
		    echo "Multiline shell steps works too"
		    ls -lah
		```
	    }
	}
    }
}
```


## step

一个`step`就像一个操作，每个step先后进行，如果有某个step失败则整个pipeline失败，当所有step成功的时候，pipeline才算是执行成功。

## retry timeout

retry自动重试操作，timeout指定操作超时时间

```java
pipeline {
    agent any
    stages {
        stage('Deploy') {
            steps {
                retry(3) {
                    sh './flakey-deploy.sh'
		}

		timeout(time: 3, unit: 'MINUTES') {
                    sh './health-check.sh'
		}
	    }
	}
    }
}
```

`timeout`和`retry`是属于Wrapper steps，就是当中可以包含其他step，包括`timeout`和`retry`，例如：

```java
pipeline {
    agent any
    stages {
        stage('Deploy') {
            steps {
	        // 自动重试五次，超过3分钟失败
                timeout(time: 3, unit: 'MINUTES') {
                    retry(5) {
                        sh './flakey-deploy.sh'
                    }
                }
            }
        }
    }
}
```

## post
post部分用来定义pipeline执行后的操作，包含总是执行、成功失败等

```java
pipeline {
    agent any
    stages {
        stage('Test') {
            steps {
                sh 'echo "Fail!"; exit 1'
	    }
	}
    }
    post {
        always {
            echo 'This will always run'
	}
	success {
            echo 'This will run only if successful'
	}
	failure {
            echo 'This will run only if failed'
	}
	unstable {
            echo 'This will run only if the sun was marked as unstable'
	}
	changed {
            echo 'This will run only if the state of the Pipeline has changed'
	    echo 'For example, if the Pipeline was previous failing but is now successful'
	}
    }
}
```

# 测试

Jenkins可以用来执行测试，也自带了junit操作

```java
pipeline {
    agent any
    stages {
        stage('Test') {
            steps {
                sh './gradlew check'
	    }
	}
    }
    post {
        always {
            junit 'build/reports/**/*.xml'
	}
    }
}
```

# 清理和通知

```java
pipeline {
    agent any
    stages {
        stage('No-op') {
            steps {
                sh 'ls'
	    }
	}
    }
    post {
        always {
            echo 'One way or another, I have finished'
	    deleteDir()
	}
	success {
            echo 'I succeeeded!'
        }
        unstable {
            echo 'I am unstable :/'
        }
        failure {
            echo 'I failed :('
        }
        changed {
            echo 'Things were different before...'
        }
    }
}
```

发送邮件通知

```java
post {
    failure {
        mail to: 'team@example.com',
	     subject: "Failed pipeline: ${currentBuild.fullDisplayName}",
	     body: "Something is wrong with ${env.BUILD_URL}"
    }
}
```

# Deployment
一个基本的持续集成管道至少要有三步：Build、Test、Deploy。

```java
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Building'
	    }
	}
	stage('Test') {
            steps {
                echo 'Testing'
	    }
	}
	stage('Deploy') {
            steps {
                echo 'Deploying'
	    }
	}
    }
}
```

请求用户输入，等到用户输入才会继续，`input`

```java
pipeline {
    agent any
    stages {
        stage('Deploy - Staging') {
            steps {
	        sh './deploy staging'
		sh './run-smoke-tests'
	    }
	}

	stage('Sanity check') {
            steps {
                input "Does the staging environment looks good?"
	    }
	}

	stage('Deploy - Production') {
            steps {
                sh './deploy production'
	    }
	}
    }
}
```

