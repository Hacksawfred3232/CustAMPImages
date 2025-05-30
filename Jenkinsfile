pipeline {
    agent any
    triggers { 
        pollSCM('H * * * *')
    }
    environment {
        DOCKER_BUILDKIT = '1'
        DOCKER_REG = 'ghcr.io/Hacksawfred3232/CustAMPImages'
    }
    stages {
        stage('Main') {
            steps {
                script {
                    def builds = [
                        ["pushto":"ampcust","dir": "SRB2", "tags":[
                            ["name":"srb2","expect-stable":true,"need-builder":false,"args":""]
                        ]
                        ],
                        ["pushto":"ampcust","dir": "SRB2Kart", "tags":[
                            ["name":"srb2kart","expect-stable":true,"need-builder":false,"args":""]
                        ]
                        ],
                        ["pushto":"ampcust","dir": "RingRacers", "tags":[
                            ["name":"ringracers","expect-stable":true,"need-builder":false,"args":""]
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
                            docker.withRegistry("https://${env.DOCKER_REG}",'githubtok') {
                                dir(builds[buildsi]["dir"]) {
                                    if (builds[buildsi]["tags"][buildsitag]["need-builder"] == true) {
                                        echo("Builder requested.")
                                        sh("docker buildx create --bootstrap --driver docker-container --name multiarch --driver-opt image=gitea.twilightcyberlycan.me.uk/tclnet/moby/buildkit:buildx-stable-1-mod")
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
                        }
                    }
                    sh("docker image prune -af")
                    sh("docker buildx prune -af")
                }
            }
        }
    }
}