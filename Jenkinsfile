pipeline {
    agent any

    environment {
        HARBOR_HOST = '192.168.56.20'
        IMAGE       = "${HARBOR_HOST}/hello-gitops/app"
        TAG         = "${env.BUILD_NUMBER}"
        BRANCH      = 'hello-gitops'
        REPO_URL    = 'https://github.com/AhmedHisham99/vprofile-project.git'
    }

    stages {
        stage('Build') {
            steps {
                sh "sed -i 's/BUILD_NUMBER_PLACEHOLDER/${TAG}/' index.html"
                sh "docker build -t ${IMAGE}:${TAG} ."
            }
        }

        stage('Push') {
            steps {
                sh '''
                    . /etc/harbor-creds.env
                    echo "$HARBOR_PASS" | docker login $HARBOR_HOST -u "$HARBOR_USER" --password-stdin
                    docker push ${IMAGE}:${TAG}
                '''
            }
        }

        // CI stops here on purpose — no kubectl. manifests/ is already
        // checked out alongside the app source (same repo/branch), so this
        // just edits the tag in place and pushes; no second clone of a
        // separate "manifests repo". Argo CD is what actually reconciles
        // the cluster from this path.
        stage('Update GitOps manifest') {
            steps {
                sh '''
                    . /etc/gitops-creds.env
                    sed -i "s#image: .*#image: ${IMAGE}:${TAG}#" manifests/deployment.yaml
                    git config user.email "jenkins@jenkins01.k8s.local"
                    git config user.name "jenkins"
                    git commit -am "hello-gitops: bump image to ${TAG}" || echo "no change to commit"
                    git push "https://${GITOPS_GIT_USER}:${GITOPS_GIT_TOKEN}@github.com/AhmedHisham99/vprofile-project.git" HEAD:${BRANCH}
                '''
            }
        }
    }

    post {
        always {
            sh 'docker logout ${HARBOR_HOST} || true'
        }
    }
}
