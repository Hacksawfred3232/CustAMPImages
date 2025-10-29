pipeline {
    agent any
    triggers { 
        pollSCM('H * * * *')
    }
    environment {
        DOCKER_BUILDKIT = '1'
        DOCKER_REG = 'ghcr.io/hacksawfred3232/custampimages'
    }
    stages {
        stage('Main') {
            steps {
                script {
                    def builds = [
                        ["pushto":"ampcust","dir": "SRB2", "tags":[
                            ["name":"v1.0.0-srb2","expect-stable":true,"need-builder":false,"args":" -t ${env.DOCKER_REG}/ampcust:srb2"]
                        ]
                        ],
                        ["pushto":"ampcust","dir": "SRB2Kart", "tags":[
                            ["name":"v1.0.0-srb2kart","expect-stable":true,"need-builder":false,"args":" -t ${env.DOCKER_REG}/ampcust:srb2kart"]
                        ]
                        ],
                        ["pushto":"ampcust","dir": "RingRacers", "tags":[
                            ["name":"v1.0.0-ringracers","expect-stable":true,"need-builder":false,"args":" -t ${env.DOCKER_REG}/ampcust:ringracers"]
                        ]
                        ],
                    ]
                    for (int buildsi = 0 ; buildsi < builds.size(); ++buildsi) {
                        for (int buildsitag = 0 ; buildsitag < builds[buildsi]["tags"].size(); ++buildsitag) {
                            if (builds[buildsi]["tags"][buildsitag]["expect-stable"] == true) {
                                finalVerName = builds[buildsi]["tags"][buildsitag]["name"]
                            } else {
                                finalVerName = "0.0.${env.BUILD_NUMBER}-${builds[buildsi]['tags'][buildsitag]['name']}"
                            }
                            def checkStatusCode = sh(returnStatus:true,script:"docker manifest inspect ${env.DOCKER_REG}/${builds[buildsi]['pushto']}:${finalVerName}")
                            if (checkStatusCode == 0) {
                                echo("Found an existing image, will not rebuild!")
                                echo("Either delete existing image, or bump version number.")
                            } else if (checkStatusCode == 1) {
                                echo("Didn't find the image in registry ${env.DOCKER_REG}, or some other status code 1 error. (Re)Building image.")
                                docker.withRegistry("https://${env.DOCKER_REG}",'githubtok') {
                                    dir(builds[buildsi]["dir"]) {
                                        if (builds[buildsi]["tags"][buildsitag]["need-builder"] == true) {
                                            echo("Builder requested.")
                                            sh("docker buildx create --bootstrap --driver docker-container --name multiarch --driver-opt image=cloudman-gitea.twilightcyberlycan.me.uk/tclnet/moby/buildkit:buildx-stable-1-mod")
                                            finalArgs = " --builder multiarch${builds[buildsi]['tags'][buildsitag]['args']}"
                                        } else {
                                            finalArgs = builds[buildsi]['tags'][buildsitag]['args']
                                        }
                                        outImg = docker.build("${env.DOCKER_REG}/${builds[buildsi]['pushto']}:${finalVerName}", ". --push${finalArgs}")
                                        if (builds[buildsi]["tags"][buildsitag]["need-builder"] == true) {
                                            sh("docker buildx rm multiarch")
                                        }
                                    }
                                }
                            } else {
                                error("Found non-zero or one status code when inspecting remote manifest. Stopping...")
                            }
                        }
                    }
                    sh("docker image prune -af")
                    sh("docker buildx prune -af")
                }
            }
        }
    }
}
