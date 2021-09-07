<template>
  <div>
    <b-row>
      <b-col cols="auto">
        <h1>Greyscaler</h1>
      </b-col>
    </b-row>

    <b-row>
      <b-col cols="12">
        <b-card bg-variant="dark" text-variant="white" title="Stap 1">
          <b-card-text>
            <b-form-group id="selectFileFG" label-for="fileSelectInput">
              <b-form-file
                id="fileSelectInput"
                v-model="selectedFile"
                class="mt-3"
                placeholder="Selecteer een afbeelding of sleep deze hier ..."
                accept="image/*"
                @input="onFileSelected"
              ></b-form-file>
            </b-form-group>
          </b-card-text>
        </b-card>
      </b-col>

      <b-col cols="12">
        <b-card bg-variant="dark" text-variant="white" class="text-center">
          <h6>Image preview:</h6>
          <div class="image-with-overflow">
            <b-img
              id="selectedImageId"
              :src="selectedImagePath"
              fluid
              alt="Geen bestand geselecteerd"
              ref="selectedImageRef"
            ></b-img>
          </div>
          <div>
            <b-alert show variant="info">
              <b-row>
                <b-col cols="6">
                  <p><strong>Pixels: </strong> {{ pixelCount }}</p>
                </b-col>
              </b-row>
              <b-row>
                <b-col cols="6">
                  <p><strong>Width: </strong> {{ imageWidth }}</p>
                  <p><strong>Heigth: </strong> {{ imageHeigth }}</p>
                </b-col>
              </b-row>
            </b-alert>
          </div>
        </b-card>
      </b-col>
    </b-row>
    <b-row>
      <b-col cols="12">
        <b-card
          class="mt-3"
          bg-variant="secondary"
          text-variant="white"
          title="Stap 2"
        >
          <div class="image-with-overflow">
            <b-img ref="greyScaledImage" :src="greyImageSrc"></b-img>
          </div>
        </b-card>
      </b-col>
    </b-row>
    <b-row>
      <b-col cols="12">
        <b-card class="mt-3" title="Stap 3">
          <div v-if="scaleRay.length > 0">
            <scale-chart :data="scaleRay" />
          </div>
        </b-card>
      </b-col>
    </b-row>
    <b-row>
      <b-col cols="12">
        <b-card class="mt-3" bg-variant="secondary" title="Stap 4">
          <div v-if="scaleRay.length > 0">
            <b-row align-h="end">
              <b-col cols="auto">
                <b-button variant="primary" @click="convertScaledRayToCSv"
                  >Save As Csv</b-button
                >
              </b-col>
            </b-row>
            <div v-for="(item, index) in scaleRay" :key="index">
              <span>Grijswaarde: {{ index }}: {{ item }}</span>
            </div>
          </div>
        </b-card>
      </b-col>
    </b-row>
  </div>
</template>

<script lang="ts">
import { Component, Prop, Ref, Vue } from "vue-property-decorator";
import { BImg } from "bootstrap-vue";
import imageGrayscale from "image-filter-grayscale";
import imageFilterCore from "image-filter-core";
import ScaleChart from "../components/ScaleChart.vue";
import getPixels from "get-pixels";
import { convertArrayToCSV } from "convert-array-to-csv";
import paper from "paper";
import { Raster, view } from "paper/dist/paper-core";

@Component({
  components: {
    ScaleChart
  }
})
export default class HomePage extends Vue {
  selectedFile: File | null = null;
  selectedImagePath: string = "";
  greyImageSrc: string = "";

  scaleRay: number[] = [];

  pixelCount: number = 0;

  get imageWidth(): number {
    if (this.selectedImageRef && this.selectedImageRef.width) {
      return this.selectedImageRef.width;
    }
    return 0;
  }
  get imageHeigth(): number {
    if (this.selectedImageRef && this.selectedImageRef.height) {
      return this.selectedImageRef.height;
    }
    return 0;
  }

  @Ref()
  selectedImageRef!: BImg;

  @Ref()
  greyScaledImage!: BImg;

  onFileSelected(file: File): void {
    this.selectedImagePath = URL.createObjectURL(file);

    // console.log(file);

    setTimeout(() => {
      console.log("Image selected");
      this.doPaperStuff(file);

      getPixels(this.selectedImageRef.src, (error: any, pixels: any) => {
        if (error) {
          console.log("Error getting pixels");
          return;
        }
        this.pixelCount = pixels.data.length;
        console.log(pixels);
      });
      this.imageToGreyScale();
    }, 1000);
  }
  doPaperStuff(file: File) {
    var raster = new Raster(this.selectedImageRef.id);
    raster.position = view.center;
    raster.scale(0.5);
    raster.rotate(45);
  }

  imageToGreyScale(): void {
    console.log(this.selectedImageRef);

    const element = this.selectedImageRef as any;
    // const element = document.createElement("img");
    // element.setAttribute("src", this.selectedImagePath);
    const canvas = document.createElement("canvas");
    const context = canvas.getContext("2d");

    if (context) {
      context.drawImage(element, 0, 0);
      const imageData = context.getImageData(
        0,
        0,
        this.selectedImageRef.width,
        this.selectedImageRef.height
      );
      imageGrayscale(imageData, 1).then((result: any) => {
        console.log(result);
        // this.greyScaledImage = new BImg();
        const canvasUrl = imageFilterCore.convertImageDataToCanvasURL(result);
        // this.greyScaledImage.setAttribute("src", canvasUrl);
        this.greyImageSrc = canvasUrl;
        this.fillGreyScaleRay(result.data);
      });
    }
  }

  fillGreyScaleRay(resultDataRay: []): void {
    const newRay = new Array<number>();
    for (let indexNewRay = 0; indexNewRay < 256; indexNewRay++) {
      newRay.push(0);
    }

    //count greyScale occurence
    for (let index = 0; index < resultDataRay.length; index++) {
      const greyScaleValue = resultDataRay[index];
      if (greyScaleValue > 256) {
        console.log("Value exeeding grayscale: " + greyScaleValue);
      }
      newRay[greyScaleValue] = newRay[greyScaleValue] + 1;
    }

    this.scaleRay = newRay;
  }

  convertScaledRayToCSv(): void {
    const jsonRay = [];
    for (let index = 0; index < this.scaleRay.length; index++) {
      const element = this.scaleRay[index];
      var propname = `value`;
      var obj: any = {};
      obj["index"] = index;
      obj[propname] = element;
      jsonRay.push(obj);
    }

    const csv = convertArrayToCSV(jsonRay);
    this.saveFile(csv);
    // this.saveFile(this.scaleRay);
  }

  saveFile(array: any): void {
    const data = array;
    // const data = JSON.stringify(array);
    const blob = new Blob([data], { type: "text/csv" });
    const e = document.createEvent("MouseEvents"),
      a = document.createElement("a");
    a.download = "grayscaler.csv";
    a.href = window.URL.createObjectURL(blob);
    a.dataset.downloadurl = ["text/csv", a.download, a.href].join(":");
    e.initEvent("click", true, false);
    // , window, 0, 0, 0, 0, 0, false, false, false, false, 0, null
    a.dispatchEvent(e);
  }
}
</script>

<style>
.image-with-overflow {
  overflow-x: auto;
  overflow-y: auto;
  height: 180px;
}
</style>
