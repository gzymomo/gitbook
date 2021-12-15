- [Jenkins搭配Nexus部署到云目标环境](https://juejin.cn/post/6942728142138277896)



**Jenkins搭配Nexus部署到云目标环境**

**1、安装Jenkins** 根据[jenkins.io/doc/pipelin…](https://jenkins.io/doc/pipeline/tour/getting-started/) 的步骤安装Jenkins

**2、安装Nexus** 按照[help.sonatype.com/repomanager…](https://help.sonatype.com/repomanager2/installing-and-running/installing) 的步骤安装Nexus

**3、为Jenkins配置Nexus插件** 根据[help.sonatype.com/integration…](https://help.sonatype.com/integrations/nexus-and-continuous-integration/nexus-repository-manager-2.x-for-jenkins) 的步骤配置Nexus插件

**4、为Jenkins添加和配置Maven Metadata for CI server插件** 这个插件添加了一个新的构建参数类型，用于解析读取存储库的工件版本。 在任何jenkins作业中，启用"This build is parametrerized"复选框，从出现的下拉菜单中选择 "List maven artifact versions"，配置您想要检索版本的工件。示例如下: ![配置nexus路径](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2d11174e39f4455f8363d1d9ec1da543~tplv-k3u1fbpfcp-zoom-1.image) **5、添加一个名为build-test的作业，用于构建项目，并将war包和tar.gz包上传到Nexus服务器。**

```
a.创建一种名为build-test的管道作业类型，然后单击“Ok”按钮保存。
复制代码
```

![Jenkins添加项目](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e5bd3629a4a24426a155de280b65c2c0~tplv-k3u1fbpfcp-zoom-1.image) b.为build-test添加一个参数 ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b02508ef2ea34d72b48eb880bac38c14~tplv-k3u1fbpfcp-zoom-1.image) c. 拷贝如下代码到pipeline脚本中

```
pipeline {
    environment {
      DEV_VERSION = "${var_version}"
      DEV_BRANCH = "branch"
   }
    agent any
    stages {
        stage('Checkout') {
            steps {
                timeout(10) {
                 checkout([$class: 'GitSCM', branches: [[name: "${DEV_BRANCH}_${DEV_VERSION}"]], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'api-portal-dex']], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'ce46885b-859c-40fe-bdaf-a6046d045044', url: 'xxx.git']]])
                checkout([$class: 'GitSCM', branches: [[name: "${DEV_BRANCH}_${DEV_VERSION}"]], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'api-mgmt-dex']], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'ce46885b-859c-40fe-bdaf-a6046d045044', url: 'xxx.git']]])
                }
            }
        }
        stage('Maven Build') {
            steps {
                timeout(10) {
                    echo 'Maven Build Start'
                    script {
                        dir('api-mgmt-dex') {
                            def mvnHome = tool name: 'M3', type: 'maven'
                            sh "${mvnHome}/bin/mvn clean org.jacoco:jacoco-maven-plugin:prepare-agent install -Dmaven.test.failure.ignore=true"
                        }
                    }
                }
            }
        }
        stage('Publish') {
        steps {
                timeout(10) {
                echo 'Publish Start'
                script {
                        def nodeHome = tool name: 'NodeJS', type: 'nodejs'
                        env.PATH = "${nodeHome}/bin:${env.PATH}"
                        sh '''cd ./api-portal-dex
                        npm install
                        npm run dist'''
                        sh 'tar -zcvf ./api-portal-dex/portal-${DEV_VERSION}.tar.gz ./api-portal-dex/dist'
                    }
                nexusPublisher nexusInstanceId: 'LocalNexus', nexusRepositoryId: 'releases', packages: [[$class: 'MavenPackage', mavenAssetList: [[classifier: '', extension: '', filePath: "./api-portal-dex/portal-${DEV_VERSION}.tar.gz"]], mavenCoordinate: [artifactId: 'portal', groupId: 'pif', packaging: 'tar.gz', version: "${DEV_VERSION}"]]]
 
                nexusPublisher nexusInstanceId: 'LocalNexus', nexusRepositoryId: 'releases', packages: [[$class: 'MavenPackage', mavenAssetList: [[classifier: '', extension: '', filePath: "./api-mgmt-dex/gateway/target/gateway-${DEV_VERSION}.war"]], mavenCoordinate: [artifactId: 'gateway', groupId: 'pif', packaging: 'war', version: "${DEV_VERSION}"]]]
 
                nexusPublisher nexusInstanceId: 'LocalNexus', nexusRepositoryId: 'releases', packages: [[$class: 'MavenPackage', mavenAssetList: [[classifier: '', extension: '', filePath: "./api-mgmt-dex/reporting/target/reporting-${DEV_VERSION}.war"]], mavenCoordinate: [artifactId: 'reporting', groupId: 'pif', packaging: 'war', version: "${DEV_VERSION}"]]]
                     
                }
            }
        }
 
    }
    post {
        success {
            emailext (
                subject: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                body: """<p>SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
                 <p>Check console output at "<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>"</p>""",
                attachLog: true,
                to: 'xxx@qq.com'
                )
        }
        failure {
            emailext (
                subject: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                body: """<p>FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
                 <p>Check console output at "<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>"</p>""",
                attachLog: true,
                to: 'xxx@qq.com'
                )
        }
    }
}
复制代码
```

**6、添加一个名为deploy-test的作业，用于下载war和tar.gz包并将它们部署到AWS服务器。**

a.创建一种名为deploy-test的管道作业类型，然后单击“Ok”按钮保存。 b.为deploy-test添加一个参数 ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8b8dc0dac42f485abee987a4e2db825d~tplv-k3u1fbpfcp-zoom-1.image)

c. 拷贝如下代码到pipeline脚本中

```
pipeline {
    environment {
      DEV_PIF_USER = "testd"
      QA_PIF_USER = "testr"
      STAGE_PIF_USER = "tests"
      PROD_PIF_USER = "testp"
 
      DEV_AMP_HOST_1 = "10.0.44.68"
      DEV_AMP_HOST_2 = "10.0.44.112"
      DEV_MPE_HOST_1 = "10.0.44.235"
      DEV_MPE_HOST_2 = "10.0.44.249"
      DEV_MRM_HOST_1 = "10.0.44.133"
      DEV_MRM_HOST_2 = "10.0.44.176"
 
      QA_AMP_HOST_1 = "10.0.33.83"
      QA_AMP_HOST_2 = "10.0.33.125"
      QA_MPE_HOST_1 = "10.0.33.234"
      QA_MPE_HOST_2 = "10.0.33.252"
      QA_MRM_HOST_1 = "10.0.33.153"
      QA_MRM_HOST_2 = "10.0.33.177"
 
      STAGE_AMP_HOST_1 = "10.0.22.69"
      STAGE_AMP_HOST_2 = "10.0.22.117"
      STAGE_MPE_HOST_1 = "10.0.22.237"
      STAGE_MPE_HOST_2 = "10.0.22.249"
      STAGE_MRM_HOST_1 = "10.0.22.155"
      STAGE_MRM_HOST_2 = "10.0.22.158"
      STAGE_MRM_HOST_3 = "10.0.22.185"
      STAGE_MRM_HOST_4 = "10.0.22.170"
 
      PROD_AMP_HOST_1 = "10.0.11.69"
      PROD_AMP_HOST_2 = "10.0.11.123"
      PROD_MPE_HOST_1 = "10.0.11.238"
      PROD_MPE_HOST_2 = "10.0.11.246"
      PROD_MRM_HOST_1 = "10.0.11.148"
      PROD_MRM_HOST_2 = "10.0.11.167"
      PROD_MRM_HOST_3 = "10.0.11.151"
      PROD_MRM_HOST_4 = "10.0.11.169"
 
      server="http://xxx:8081/nexus/service/local/artifact/maven/content"
      group="test"
      artifact_gateway="backend"
      artifact_reporting="reporting"
      artifact_portal="portal"
      version="${var_version}"
 
      repo="${var_repo}"
       
      nexus_user="xxx"
      nexus_passwd="xxx"
 
   }
    agent any
    stages {
         
        stage('Download Artifact From Nexus') {
            steps {
                timeout(10) {
                    echo 'Download Artifact From Nexus Start'
                    script {
                       sh '''
                              curl -o "gateway-"${version}".war" -X GET -u $nexus_user:$nexus_passwd $server"?g="$group"&a=gateway&v="$version"&r="releases"&e=war"
                              curl -o "reporting-"${version}".war" -X GET -u $nexus_user:$nexus_passwd $server"?g="$group"&a=reporting&v="$version"&r="releases"&e=war"
                              curl -o "portal-"${version}".tar.gz" -X GET -u $nexus_user:$nexus_passwd $server"?g="$group"&a=portal&v="$version"&r="releases"&e=tar.gz"
                          '''
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                timeout(10) {
                    echo 'Deploy Start'
                    script {
                         sh '''
                                tar -zxvf ./portal-${version}.tar.gz
                            '''
                    }
                    sshagent(credentials: ['22630c00-9964-425a-bd57-26a673806d17']) {
 
                        sh '''
                              if [ ${var_repo} = "dev" ]
                              then
                                  echo 'dev env start deploying'
                                  scp -o StrictHostKeyChecking=no -r ./api-portal-dex/dist/* ${DEV_PIF_USER}@${DEV_AMP_HOST_1}:/app/pif/html
                                  scp -o StrictHostKeyChecking=no -r ./api-portal-dex/dist/* ${DEV_PIF_USER}@${DEV_AMP_HOST_2}:/app/pif/html
 
                                  ssh -o StrictHostKeyChecking=no -l ${DEV_PIF_USER} ${DEV_AMP_HOST_1} sudo /etc/init.d/dex_pifamp restart
                                  ssh -o StrictHostKeyChecking=no -l ${DEV_PIF_USER} ${DEV_AMP_HOST_2} sudo /etc/init.d/dex_pifamp restart
 
                                  scp -o StrictHostKeyChecking=no ./gateway-${version}.war ${DEV_PIF_USER}@${DEV_MPE_HOST_1}:/home/${DEV_PIF_USER}/gateway.war
                                  scp -o StrictHostKeyChecking=no ./gateway-${version}.war ${DEV_PIF_USER}@${DEV_MPE_HOST_2}:/home/${DEV_PIF_USER}/gateway.war
 
                                  scp -o StrictHostKeyChecking=no ./reporting-${version}.war ${DEV_PIF_USER}@${DEV_MRM_HOST_1}:/home/${DEV_PIF_USER}/reporting.war
                                  scp -o StrictHostKeyChecking=no ./reporting-${version}.war ${DEV_PIF_USER}@${DEV_MRM_HOST_2}:/home/${DEV_PIF_USER}/reporting.war
 
                                  ssh -o StrictHostKeyChecking=no -l ${DEV_PIF_USER} ${DEV_MPE_HOST_1} /home/${DEV_PIF_USER}/.script/deploy.sh
                                  ssh -o StrictHostKeyChecking=no -l ${DEV_PIF_USER} ${DEV_MPE_HOST_2} /home/${DEV_PIF_USER}/.script/deploy.sh
 
                                  ssh -o StrictHostKeyChecking=no -l ${DEV_PIF_USER} ${DEV_MRM_HOST_1} /home/${DEV_PIF_USER}/.script/deploy.sh
                                  ssh -o StrictHostKeyChecking=no -l ${DEV_PIF_USER} ${DEV_MRM_HOST_2} /home/${DEV_PIF_USER}/.script/deploy.sh
 
                              elif [ ${var_repo} = "qa" ]
                              then
                                  echo 'qa env start deploying'
                                  scp -o StrictHostKeyChecking=no -r ./api-portal-dex/dist/* ${QA_PIF_USER}@${QA_AMP_HOST_1}:/app/pif/html
                                  scp -o StrictHostKeyChecking=no -r ./api-portal-dex/dist/* ${QA_PIF_USER}@${QA_AMP_HOST_2}:/app/pif/html
 
                                  ssh -o StrictHostKeyChecking=no -l ${QA_PIF_USER} ${QA_AMP_HOST_1} sudo /etc/init.d/dex_pifamp restart
                                  ssh -o StrictHostKeyChecking=no -l ${QA_PIF_USER} ${QA_AMP_HOST_2} sudo /etc/init.d/dex_pifamp restart
 
                                  scp -o StrictHostKeyChecking=no ./gateway-${version}.war ${QA_PIF_USER}@${QA_MPE_HOST_1}:/home/${QA_PIF_USER}/gateway.war
                                  scp -o StrictHostKeyChecking=no ./gateway-${version}.war ${QA_PIF_USER}@${QA_MPE_HOST_2}:/home/${QA_PIF_USER}/gateway.war
 
                                  scp -o StrictHostKeyChecking=no ./reporting-${version}.war ${QA_PIF_USER}@${QA_MRM_HOST_1}:/home/${QA_PIF_USER}/reporting.war
                                  scp -o StrictHostKeyChecking=no ./reporting-${version}.war ${QA_PIF_USER}@${QA_MRM_HOST_2}:/home/${QA_PIF_USER}/reporting.war
 
                                  ssh -o StrictHostKeyChecking=no -l ${QA_PIF_USER} ${QA_MPE_HOST_1} /home/${QA_PIF_USER}/.script/deploy.sh
                                  ssh -o StrictHostKeyChecking=no -l ${QA_PIF_USER} ${QA_MPE_HOST_2} /home/${QA_PIF_USER}/.script/deploy.sh
                                   
                                  ssh -o StrictHostKeyChecking=no -l ${QA_PIF_USER} ${QA_MRM_HOST_1} /home/${QA_PIF_USER}/.script/deploy.sh
                                  ssh -o StrictHostKeyChecking=no -l ${QA_PIF_USER} ${QA_MRM_HOST_2} /home/${QA_PIF_USER}/.script/deploy.sh
 
                              elif [ ${var_repo} = "stage" ]
                              then
                                  echo 'stage env start deploying'
                                  scp -o StrictHostKeyChecking=no -r ./api-portal-dex/dist/* ${STAGE_PIF_USER}@${STAGE_AMP_HOST_1}:/app/pif/html
                                  scp -o StrictHostKeyChecking=no -r ./api-portal-dex/dist/* ${STAGE_PIF_USER}@${STAGE_AMP_HOST_2}:/app/pif/html
 
                                  ssh -o StrictHostKeyChecking=no -l ${STAGE_PIF_USER} ${STAGE_AMP_HOST_1} sudo /etc/init.d/dex_pifamp restart
                                  ssh -o StrictHostKeyChecking=no -l ${STAGE_PIF_USER} ${STAGE_AMP_HOST_2} sudo /etc/init.d/dex_pifamp restart
 
                                  scp -o StrictHostKeyChecking=no ./gateway-${version}.war ${STAGE_PIF_USER}@${STAGE_MPE_HOST_1}:/home/${STAGE_PIF_USER}/gateway.war
                                  scp -o StrictHostKeyChecking=no ./gateway-${version}.war ${STAGE_PIF_USER}@${STAGE_MPE_HOST_2}:/home/${STAGE_PIF_USER}/gateway.war
 
                                  scp -o StrictHostKeyChecking=no ./reporting-${version}.war ${STAGE_PIF_USER}@${STAGE_MRM_HOST_1}:/home/${STAGE_PIF_USER}/reporting.war
                                  scp -o StrictHostKeyChecking=no ./reporting-${version}.war ${STAGE_PIF_USER}@${STAGE_MRM_HOST_2}:/home/${STAGE_PIF_USER}/reporting.war
                                  scp -o StrictHostKeyChecking=no ./reporting-${version}.war ${STAGE_PIF_USER}@${STAGE_MRM_HOST_3}:/home/${STAGE_PIF_USER}/reporting.war
                                  scp -o StrictHostKeyChecking=no ./reporting-${version}.war ${STAGE_PIF_USER}@${STAGE_MRM_HOST_4}:/home/${STAGE_PIF_USER}/reporting.war
 
                                  ssh -o StrictHostKeyChecking=no -l ${STAGE_PIF_USER} ${STAGE_MPE_HOST_1} /home/${STAGE_PIF_USER}/.script/deploy.sh
                                  ssh -o StrictHostKeyChecking=no -l ${STAGE_PIF_USER} ${STAGE_MPE_HOST_2} /home/${STAGE_PIF_USER}/.script/deploy.sh
                                   
                                  ssh -o StrictHostKeyChecking=no -l ${STAGE_PIF_USER} ${STAGE_MRM_HOST_1} /home/${STAGE_PIF_USER}/.script/deploy.sh
                                  ssh -o StrictHostKeyChecking=no -l ${STAGE_PIF_USER} ${STAGE_MRM_HOST_2} /home/${STAGE_PIF_USER}/.script/deploy.sh
                                  ssh -o StrictHostKeyChecking=no -l ${STAGE_PIF_USER} ${STAGE_MRM_HOST_3} /home/${STAGE_PIF_USER}/.script/deploy.sh
                                  ssh -o StrictHostKeyChecking=no -l ${STAGE_PIF_USER} ${STAGE_MRM_HOST_4} /home/${STAGE_PIF_USER}/.script/deploy.sh
 
                              elif [ ${var_repo} = "prod" ]
                              then
                                  echo 'prod env start deploying'
                                  scp -o StrictHostKeyChecking=no -r ./api-portal-dex/dist/* ${PROD_PIF_USER}@${PROD_AMP_HOST_1}:/app/pif/html
                                  scp -o StrictHostKeyChecking=no -r ./api-portal-dex/dist/* ${PROD_PIF_USER}@${PROD_AMP_HOST_2}:/app/pif/html
 
                                  ssh -o StrictHostKeyChecking=no -l ${PROD_PIF_USER} ${PROD_AMP_HOST_1} sudo /etc/init.d/dex_pifamp restart
                                  ssh -o StrictHostKeyChecking=no -l ${PROD_PIF_USER} ${PROD_AMP_HOST_2} sudo /etc/init.d/dex_pifamp restart
 
                                  scp -o StrictHostKeyChecking=no ./gateway-${version}.war ${PROD_PIF_USER}@${PROD_MPE_HOST_1}:/home/${PROD_PIF_USER}/gateway.war
                                  scp -o StrictHostKeyChecking=no ./gateway-${version}.war ${PROD_PIF_USER}@${PROD_MPE_HOST_2}:/home/${PROD_PIF_USER}/gateway.war
 
                                  scp -o StrictHostKeyChecking=no ./reporting-${version}.war ${PROD_PIF_USER}@${PROD_MRM_HOST_1}:/home/${PROD_PIF_USER}/reporting.war
                                  scp -o StrictHostKeyChecking=no ./reporting-${version}.war ${PROD_PIF_USER}@${PROD_MRM_HOST_2}:/home/${PROD_PIF_USER}/reporting.war
                                  scp -o StrictHostKeyChecking=no ./reporting-${version}.war ${PROD_PIF_USER}@${PROD_MRM_HOST_3}:/home/${PROD_PIF_USER}/reporting.war
                                  scp -o StrictHostKeyChecking=no ./reporting-${version}.war ${PROD_PIF_USER}@${PROD_MRM_HOST_4}:/home/${PROD_PIF_USER}/reporting.war
 
                                  ssh -o StrictHostKeyChecking=no -l ${PROD_PIF_USER} ${PROD_MPE_HOST_1} /home/${PROD_PIF_USER}/.script/deploy.sh
                                  ssh -o StrictHostKeyChecking=no -l ${PROD_PIF_USER} ${PROD_MPE_HOST_2} /home/${PROD_PIF_USER}/.script/deploy.sh
                                   
                                  ssh -o StrictHostKeyChecking=no -l ${PROD_PIF_USER} ${PROD_MRM_HOST_1} /home/${PROD_PIF_USER}/.script/deploy.sh
                                  ssh -o StrictHostKeyChecking=no -l ${PROD_PIF_USER} ${PROD_MRM_HOST_2} /home/${PROD_PIF_USER}/.script/deploy.sh
                                  ssh -o StrictHostKeyChecking=no -l ${PROD_PIF_USER} ${PROD_MRM_HOST_3} /home/${PROD_PIF_USER}/.script/deploy.sh
                                  ssh -o StrictHostKeyChecking=no -l ${PROD_PIF_USER} ${PROD_MRM_HOST_4} /home/${PROD_PIF_USER}/.script/deploy.sh
 
                              else
                                  echo "The version format is error!"
                              fi
 
                           '''
                    }
                }
            }
        }
    }
    post {
        success {
            emailext (
                subject: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                body: """<p>SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
                 <p>Check console output at "<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>"</p>""",
                attachLog: true,
                to: 'xxx@qq.com'
                )
        }
        failure {
            emailext (
                subject: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                body: """<p>FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
                 <p>Check console output at "<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>"</p>""",
                attachLog: true,
                to: 'xxx@qq.com'
                )
        }
    }
}
复制代码
```

d.带参数构建“deploy-test”作业 ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/51815448083a400ab47b7db81147691a~tplv-k3u1fbpfcp-zoom-1.image)

