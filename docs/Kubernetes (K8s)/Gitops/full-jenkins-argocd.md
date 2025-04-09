# Toàn luồng Jenkins + Argocd + Github + Docker hub trên k8s

## 1. Giới thiệu

Trong bài viết trước Jenkins đã lấy source từ github sau đó dùng kaniko để build Dockerfile có trong source, sau đó đẩy image lên docker hub, và argocd dùng image của trên docker hub để deployment ứng dụng lên k8s. Bài viết này mình sẽ giới thiệu kết hợp toàn trình luồng gitops: Jenkins + Argocd + Github + Docker hub trên k8s.

Ý tưởng là dùng Jenkins lấy source sau đó dùng kaniko build Dockerfile và đẩy lên docker hub lên version mới, sau đó chỉnh version của tag image trong values trong repo config. Argocd sẽ theo dõi thay đổi của repo config để deployment ứng dụng lên k8s


## 2. Cài đặt

### 2.1. Cấu hình jenkins

Mã nguồn github: [https://github.com/vanphuoc9/complete-prodcution-e2e-pipeline.git](https://github.com/vanphuoc9/complete-prodcution-e2e-pipeline.git)

Cấu hình jenkins giống [bài viết Jenkins Build Docker image và push lên Docker Hub](jenkins-build-image.md), chỉ cần thay đổi lại cấu hình Script Path sử dụng "Jenkinsfile-k8s-full" trong [https://github.com/vanphuoc9/complete-prodcution-e2e-pipeline.git](https://github.com/vanphuoc9/complete-prodcution-e2e-pipeline.git)

![gitops](images/image1.png)


```pipeline title="Jenkinsfile-k8s-full" linenums="1"
def appConfigRepo = 'https://github.com/vanphuoc9/complete-prodcution-e2e-pipeline-config.git'
def appConfigBranch = 'main'

def helmRepo = "complete-prodcution-e2e-pipeline-config"
def helmValueFile = "values.yaml"

pipeline {
    agent any

    environment {
        BUILD_NUMBER = "${env.BUILD_NUMBER}"
        version = "v1.${BUILD_NUMBER}"
        DOCKER_IMAGE_NAME = "thaiphuoc1997/testk8s"
        DOCKER_IMAGE = "${DOCKER_IMAGE_NAME}:${version}"
    }

    options {
        skipDefaultCheckout()
    }

    parameters {
        string(name: 'GIT_URL', defaultValue: 'https://github.com/vanphuoc9/complete-prodcution-e2e-pipeline.git', description: 'The URL of the source Git repository to use.')
        string(name: 'GIT_BRANCH', defaultValue: 'main', description: 'The branch in the source Git repository to use.')
    }

    stages {
        stage("Checkout") {
            steps {
                checkout(changelog: false, poll: false, scm: [
                    $class: 'GitSCM',
                    branches: [[name: params.GIT_BRANCH]],
                    doGenerateSubmoduleConfigurations: false,
                    submoduleCfg: [],
                    userRemoteConfigs: [[url: params.GIT_URL]],
                ])
                stash name: 'sources', includes: '**', excludes: '**/.git,**/.git/**'
            }
        }

        stage("Build And Push Docker Image") {
            agent {
                label 'docker-build'
            }
            steps {
                unstash 'sources'
                container(name: 'kaniko') {
                    sh '/kaniko/executor --context=`pwd` --dockerfile=`pwd`/Dockerfile --destination=${DOCKER_IMAGE}'
                }
            }
        }

        stage('Update version in helm-chart') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                    sh """#!/bin/bash
                        set -e

                        [[ -d ${helmRepo} ]] && rm -rf ${helmRepo}

                        git clone ${appConfigRepo} --branch ${appConfigBranch}
                        cd ${helmRepo}

                        # Cập nhật tag với version
                        sed -i "s|  tag: .*|  tag: \\\"${version}\\\"|" ${helmValueFile}

                        git config user.name "vanphuoc9"
                        git config user.email "thaiphuoc1997@gmail.com"

                        git add .
                        git commit -m "Update to version ${version}" || echo "No changes to commit"
                        git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/vanphuoc9/complete-prodcution-e2e-pipeline-config.git

                        cd ..
                        rm -rf ${helmRepo}
                    """
                }
            }
        }
    }
}
```

### 2.2. Cấu hình Argocd
Cấu hình Argocd giống [bài viết Cấu hình luồng Argocd deploy lên k8s](argocd-deploy.md)

## 3. Tài liệu tham khảo

- [Xây dựng luồng Gitops với Jenkins và ArgoCD](https://viblo.asia/p/k8s-phan-21-xay-dung-luong-gitops-voi-jenkins-va-argocd-y3RL1aX7Lao#_tao-access-key-de-truy-cap-github-15)