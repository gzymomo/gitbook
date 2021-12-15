微信公众号：DevOps云学堂：Jenkins声明式管道语法快速参考指南

Jenkins管道使用户能够构建完整的持续交付（CD）管道，并作为其应用程序代码的一部分。构建，测试和交付步骤成为应用程序本身的一部分，存储在Jenkinsfile中。声明式管道语法提供了一个简单的预定义层次结构，以使所有经验级别的用户都可以访问管道和相关的Jenkinsfiles的创建。最简单的形式是，管道在代理上运行并包含阶段，而每个阶段都包含定义特定操作的步骤。

```
pipeline {
  agent {
      label ''
  }
  stages {
    stage('Build') {
      steps{
     sh 'mvn install'
    }
   }
  }
}
```

此外，声明式管道语法还提供以简单的格式控制管道执行环境的各个方面的能力。例如，使用Maven在Docker容器中构建Java应用程序，该容器仅存档和测试"Master"分支，并在六个小时后超时。

```
pipeline {
  agent {
    docker {
      label ‘docker-node’
      image ‘maven’
      args ‘-v /tmp:/tmp -p 80:80’
    }
  }
  environment {
    GIT_COMMITTER_NAME = ‘jenkins’
  }
  options {
    timeout(6, HOURS)
  }
  stages {
    stage(‘Build’) {
      steps {
         sh ‘mvn clean install’
      }
    }
    
    stage(‘Archive’) {
      when {
       branch ‘*/master’
      }
      steps {
        archive ‘*/target/**/*’
        junit ‘*/target/surefire-reports/*.xml’
      }
    }
  }

  post {
    always {
       deleteDir()
    }
  }
}
```

## 声明式管道语法（必要） 

**pipeline**:  定义一条Jenkins管道。

**agent：** 定义用于执行管道阶段的代理节点。

- label:  Jenkins node节点的标签

- docker: 使用Docker类型的节点

- - image：指定docker镜像。
  - args：docker容器所接收的参数。

**stages**:  流水线所包含的阶段和步骤。

**stage：** 流水线中的一个阶段

- steps：一个构建步骤：sh,bat,timeout,echo,archive,junit..

- - parallel: 并行步骤（可选）。
  - script：执行一个脚本块。

- when: 阶段运行的条件，例如根据分支、变量判断。

- agent, environment,tools and post

------

## 声明式管道语法（可选） 

**environment：** 定义管道运行时环境变量。

**options:** 定义管道运行时选项。

- skipDefaultCheckout：禁止自动checkout SCM。
- timeout：指定管道的运行超时时间。
- buildDiscarder：丢弃旧版本历史构建。
- disableConcurrentBuilds: 禁止并行运行。

**tools** ：预先安装的工具可用路径。

**triggers**:  管道的调度，构建触发器。

**parameters**：定义管道的运行时参数。

**post**：定义当管道运行后的操作。

- always：总是执行。
- success：管道状态为success执行。
- failure：管道状态为failed时执行。