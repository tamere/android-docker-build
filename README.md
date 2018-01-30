# Android Docker Build

This Dockerfile installs all the necessary pacakges to build a Docker image
that can be used in your build pipeline. We currently use it with Bitbucket
Pipelines but it should work with any CI system that support docker images.

DockerHub: https://hub.docker.com/r/tamerescrl/android-docker-build/

## Build and publish to DockerHub

    $ docker build . --no-cache -t tamerescrl/android-docker-build
    $ docker push tamerescrl/android-docker-build:latest

## Usage in Bitbucket Pipeline

Example:

    pipelines:
    default:
        - step:
            name: Build
            image: tamerescrl/android-docker-build:latest
            caches:
                - gradle
                - maven
            script:
                # Submodule init
                - git submodule update --init

                # Setup env vars
                - echo "BITBUCKET_USERNAME=${BITBUCKET_USERNAME}">>gradle.properties
                - echo "BITBUCKET_PASSWORD=${BITBUCKET_PASSWORD}">>gradle.properties
                - echo "SIGN_KEY_ALIAS=${HYTUNE_KEY_ALIAS}">>gradle.properties
                - echo "SIGN_KEYSTORE_PASSWORD=${HYTUNE_KEYSTORE_PASSWORD}">>gradle.properties

                # Build itself
                - ./gradlew :app:assembleDebug -PdisablePreDex
                - ./gradlew :app:assembleAndroidTest -PdisablePreDex

                # Artifacts 
                - mkdir artifacts
                - cp app/build/outputs/apk/internal/debug/app-internal-debug.apk artifacts/apk
                - cp app/build/outputs/apk/androidTest/internal/debug/app-internal-debug-androidTest.apk artifacts/test_apk
                artifacts:
                    - artifacts/**