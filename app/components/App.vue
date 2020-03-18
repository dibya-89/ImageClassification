<template>
    <Page>
        <ActionBar title="Home" />
        <ScrollView>
            <StackLayout class="home-panel">
                <!--Add your page content here-->
                <Image v-if="pictureFromCamera" class="mainImage"
                    :src="pictureFromCamera" />
                <Button class="button" text="Take Picture"
                    @tap="takePicture" />
                <Label v-if="predictName" class="data" textWrap="true"
                    :text="predictName" height="50" width="350"
                    backgroundColor="#8faf88" />

            </StackLayout>
        </ScrollView>
    </Page>
</template>

<script>
    import * as camera from "nativescript-camera";
    import * as bghttp from "nativescript-background-http";
    const imageSourceModule = require("tns-core-modules/image-source");
    const fileSystemModule = require("tns-core-modules/file-system");

    export default {
        data() {
            return {
                pictureFromCamera: null,
                predictName: null
            };
        },
        methods: {
            takePicture() {
                camera.requestPermissions();
                camera
                    .takePicture({
                        width: 108,
                        height: 162,
                        keepAspectRatio: true
                    })
                    .then(picture => {
                        this.pictureFromCamera = picture;
                        const source = new imageSourceModule
                    .ImageSource();
                        source.fromAsset(picture).then(imageSource => {
                            const folder = fileSystemModule
                                .knownFolders.documents()
                                .path;
                            const fileName = "picture.png";
                            const path = fileSystemModule.path
                                .join(
                                    folder,
                                    fileName
                                );
                            const picsaved = imageSource
                                .saveToFile(path, "png");
                            if (picsaved) {
                                console.log("Saved");
                                var session = bghttp.session(
                                    "image-upload");
                                var request = {
                                    url: "https://dibs.cognitiveservices.azure.com/customvision/v3.0/Prediction/2073859e-43aa-418d-bb23-183ce3c35985/classify/iterations/Iteration1/image",
                                    method: "POST",
                                    headers: {
                                        "Content-Type": "application/octet-stream",
                                        "Prediction-Key": "3101822b829840af8dd1daf2dd8cae25"
                                    }
                                };
                                try {
                                    var task = session.uploadFile(
                                        path, request);
                                } catch (err) {
                                    console.log(err);
                                }
                                task.on("responded", data => {
                                    const result = JSON
                                        .parse(data.data)
                                        .predictions[0]
                                        .tagName;
                                    this.predictName =
                                        result;
                                });
                            } else {
                                console.log("Failed");
                            }
                        });
                    })
                    .catch(err => {
                        console.log("Error: " + err.message);
                    });
            }
        }
    };
</script>

<style scoped>
    .home-panel {
        vertical-align: center;
        font-size: 20;
        margin: 15;
    }

    .page {
        background-image: linear-gradient(to right, #4D7C8A, #7F9C96);
    }

    .actionBar {
        background-color: #1B4079;
        color:
            #ffffff;
    }

    .mainImage {
        margin: 200px;
        margin-bottom: 25px;
        margin-top: 25px;
        border-radius: 15px;
        padding: 5rem;
        object-fit:
            contain;
    }

    .button {
        margin-bottom: 50px;
    }

    .data {
        border-radius: 15px;
        font-size: 22;
        font-weight: bold;
        text-align: center;
    }
</style>