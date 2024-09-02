pipeline{
    agent{
        node{
            label 'maven'
        }
    }
    enviroment{
        RHT_OCP$_DEV_USER = 'ctmhlo'
        DEPLOYMENT_STAGE = 'shopping-cart-stage'
        DEPLOYMENT_PRODUCTION = 'shopping-cart-production'
    }
    stages{
        stage('Tests'){
            steps{
                sh'./mvnw clean test'
            }
        }
        stage('Package'){
            steps{
                sh '''
                    ./mvnw package -DskipTests \
                    -Dquarkus.package.type=uber-jar
                '''
                archiveArtifacts 'target/*.jar'
            }
        }
        stage('Build Image'){
            environment { QUAY = credentials('QUAY_USER')}
            steps{
                sh '''
                    ./mvnw quarkus:add-extension \
                    -Dextensions="kubernetes,container-image-jib"
                '''
                sh '''
                    ./mvnw package -DskipTests \
                    -Dquarkus.jib.base-jvm-image=quay.io/redhattraining/do400-java-alpine-openjdk11-jre \
                    -Dquarkus.container-image.build=true \
                    -Dquarkus.container-image.registry=quay.io \
                    -Dquarkus.container-image.group=$QUAY_USR \
                    -Dquarkus.container-image.name=do400-deploying-environments \
                    -Dquarkus.container-image.username=$QUAY_USR \
                    -Dquarkus.container-image.password="$QUAY_PSW" \
                    -Dquarkus.container-image.tag=build-${BUILD_NUMBER}
                    -Dquarkus.container-image.push=true
                '''
            }
        }
        stage('Deploy-Stage'){
            environment{
                APP_NAMESPACE = "shopping-cart-stage"
                QUAY = credentials('QUAY_USER')
            }
            steps{
                sh '''
                    oc set image \
                    deployment ${DPLOYMENT_STAGE} \
                    shopping-cart-stage=quay.io/${QUAY_USER}/do400-deploying-enviroenments:build-${BUILD_NUMBER} \
                    -n ${APP_NAMESPACE} --record
                '''
            }
        }
    }
}