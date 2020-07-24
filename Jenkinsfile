def phone(String ip, String cmd, String step_label="") {
  def label_txt = step_label == null || step_label.isEmpty() ? cmd : step_label;
  env.TEST_DIR = "/data/openpilot"

  def ssh_cmd = """
                printenv
                ssh -o StrictHostKeyChecking=no -i selfdrive/test/id_rsa -p 8022 root@${ip} '/usr/bin/bash -sl <<EOF
                export CI=1
                export TEST_DIR="${env.TEST_DIR}"
                export GIT_BRANCH="${env.GIT_BRANCH}"
                export GIT_COMMIT="${env.GIT_COMMIT}"
                cd ${TEST_DIR} || true
                set -ex
                ${cmd}
EOF'"""

  sh label: "phone: ${label_txt}",
     script: ssh_cmd
}

def setup_environment(String ip) {
  phone(ip, 'echo "$(cat selfdrive/test/setup_ci.sh)"', "git checkout")
}

pipeline {
  agent {
    docker {
      image 'python:3.7.3'
      args '--user=root'
    }
  }
  environment {
    COMMA_JWT = credentials('athena-test-jwt')
  }

  stages {

    stage('Release Build') {
      when {
        branch 'devel-staging'
      }
      steps {
        lock(resource: "", label: 'eon-build', inversePrecedence: true, variable: 'device_ip', quantity: 1){
          timeout(time: 60, unit: 'MINUTES') {
            dir(path: 'selfdrive/test') {
              sh 'pip install paramiko'
              sh 'python phone_ci.py "cd release && PUSH=1 ./build_release2.sh"'
            }
          }
        }
      }
    }

    stage('On-device Tests') {
      when {
        not {
          anyOf {
            branch 'master-ci'; branch 'devel'; branch 'devel-staging'; branch 'release2'; branch 'release2-staging'; branch 'dashcam'; branch 'dashcam-staging'
          }
        }
      }

      parallel {

        /*
        stage('Build') {
          environment {
            CI_PUSH = "${env.BRANCH_NAME == 'master' ? 'master-ci' : ''}"
          }

          steps {
            lock(resource: "", label: 'eon', inversePrecedence: true, variable: 'device_ip', quantity: 1){
              timeout(time: 60, unit: 'MINUTES') {
                dir(path: 'selfdrive/test') {
                  sh 'pip install paramiko'
                  sh 'python phone_ci.py "cd release && ./build_devel.sh"'
                }
              }
            }
          }
        }
        */

        stage('Replay Tests') {
          steps {
            lock(resource: "", label: 'eon2', inversePrecedence: true, variable: 'device_ip', quantity: 1){
              timeout(time: 60, unit: 'MINUTES') {
                setup_environment(device_ip)
                //phone(device_ip, "cd selfdrive/test/process_replay && ./camera_replay.py", "camerad/modeld replay")
              }
            }
          }
        }

        /*
        stage('HW Tests') {
          steps {
            lock(resource: "", label: 'eon', inversePrecedence: true, variable: 'device_ip', quantity: 1){
              timeout(time: 60, unit: 'MINUTES') {
                dir(path: 'selfdrive/test') {
                  sh 'pip install paramiko'
                  sh 'python phone_ci.py "SCONS_CACHE=1 scons -j3 cereal/ && \
                                          nosetests -s selfdrive/test/test_sounds.py && \
                                          nosetests -s selfdrive/boardd/tests/test_boardd_loopback.py"'
                }
              }
            }
          }
        }
        */

      }
    }

  }
}
