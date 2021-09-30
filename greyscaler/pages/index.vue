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
            <b-row>
              <b-col cols="4"
                ><b-form-group
                  id="inputZSCale"
                  label-for="zScaleInput"
                  label="Z-schaal"
                >
                  <b-form-input
                    id="zScaleInput"
                    type="number"
                    v-model="zScale"
                    class="mt-3"
                    placeholder="0-255"
                    min="0"
                    max="255"
                  ></b-form-input> </b-form-group
              ></b-col>
            </b-row>
            <b-row>
              <b-col>
                <b-button @click="onCalculateClicked">Berekenen</b-button>
              </b-col>
            </b-row>
          </b-card-text>
        </b-card>
      </b-col>

      <b-col cols="12">
        <b-card bg-variant="dark" text-variant="white" class="text-center">
          <h6>Image preview:</h6>
          <div class="image-previeww-rapper">
            <div class="image-with-overflow ">
              <b-img
                class="zoom"
                id="selectedImageId"
                :src="selectedImagePath"
                fluid
                alt="Geen bestand geselecteerd"
                ref="selectedImageRef"
              ></b-img>
            </div>

            <div class="image-with-overflow mt-3">
              <h6>Scaled image impression:</h6>
              <!-- <img ref="greyScaledImage" /> -->
              <canvas class="zoom" ref="greyScaledImage" />
            </div>
          </div>
        </b-card>
      </b-col>
    </b-row>
    <b-row>
      <b-col cols="12">
        <b-card class="mt-3" bg-variant="secondary" title="Csv">
          <div v-if="matrixRay.length > 0">
            <b-row align-h="end">
              <b-col cols="auto">
                <b-button variant="primary" @click="convertScaledRayToCSv"
                  >Save As Csv</b-button
                >
              </b-col>
            </b-row>
            <!-- <div v-for="(item, index) in scaleRay" :key="index">
              <span>Grijswaarde: {{ index }}: {{ item }}</span>
            </div> -->
          </div>
        </b-card>
      </b-col>
    </b-row>
  </div>
</template>

<script lang="ts">
import { Component, Prop, Ref, Vue } from "vue-property-decorator";
import { BImg } from "bootstrap-vue";
import ScaleChart from "../components/ScaleChart.vue";
import { convertArrayToCSV } from "convert-array-to-csv";

interface IPixelColorIndexes {
  red: number;
  green: number;
  blue: number;
  alpha: number;
  scaledGreyValue: number;
}

@Component({
  components: {
    ScaleChart
  }
})
export default class HomePage extends Vue {
  selectedFile: File | null = null;
  selectedImagePath: string = "";
  pixelCount: number = 0;
  zScale: number = 255;
  matrixRay: Array<Array<IPixelColorIndexes>> = [[]];
  originalImageData: Uint8ClampedArray | null = null;
  greyImageData: Uint8ClampedArray | null = null;

  @Ref()
  selectedImageRef!: HTMLImageElement;

  @Ref()
  greyScaledImage!: HTMLCanvasElement;

  onFileSelected(file: File): void {
    this.selectedImagePath = URL.createObjectURL(file);
    console.log("Image selected");
    this.resetAfterSelect();
  }

  onCalculateClicked(): void {
    setTimeout(() => {
      this.calculateImageProps();
    }, 1000);
  }

  resetAfterSelect(): void {
    if (this.greyScaledImage) {
      const context = this.greyScaledImage.getContext("2d");
      if (context) {
        context.clearRect(
          0,
          0,
          this.greyScaledImage.width,
          this.greyScaledImage.height
        );
      }
    }

    this.matrixRay = [];
    this.originalImageData = null;
    this.greyImageData = null;
  }

  calculateImageProps(): void {
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
      console.log(imageData);
      this.originalImageData = imageData.data;

      this.setMatrixRay(imageData);
      setTimeout(() => {
        this.setScaledImage();
      }, 500);
    }
  }

  setMatrixRay(imageData: ImageData): void {
    const matrixRay = [];
    for (let yCoord = 0; yCoord < imageData.height; yCoord++) {
      const rowLine = [];

      for (let xCoord = 0; xCoord < imageData.width; xCoord++) {
        const coordColorIndex: IPixelColorIndexes = this.getColorIndicesForCoord(
          xCoord,
          yCoord,
          imageData
        );
        coordColorIndex.scaledGreyValue = Math.floor(
          (coordColorIndex.red / 255) * this.zScale // should be able to take any of the rgb colors because the picture is already a greyscaled image(= all rgb values are equal)
        );
        rowLine.push(coordColorIndex);
      }

      matrixRay.push(rowLine);
    }
    console.log(matrixRay);
    this.matrixRay = matrixRay;
  }

  getColorIndicesForCoord(
    xCoord: number,
    yCoord: number,
    imageData: ImageData
  ): IPixelColorIndexes {
    var pixelFirstPosition: number =
      yCoord * (imageData.width * 4) + xCoord * 4; //aka the red index of the pixel

    return {
      red: imageData.data[pixelFirstPosition],
      green: imageData.data[pixelFirstPosition + 1],
      blue: imageData.data[pixelFirstPosition + 2],
      alpha: imageData.data[pixelFirstPosition + 3],
      scaledGreyValue: 0
    };
  }

  setScaledImage(): void {
    const greyScaleCanvas = this.greyScaledImage;
    greyScaleCanvas.width = this.selectedImageRef.width;
    greyScaleCanvas.height = this.selectedImageRef.width;
    console.log("greyScaleCanvas");
    console.log(greyScaleCanvas);

    const context = greyScaleCanvas.getContext("2d");
    // Note: requires Matrixray to be filled
    if (context) {
      // draw an image on the greyScale element
      var image = new Image();
      image.width = this.selectedImageRef.width;
      image.height = this.selectedImageRef.width;
      context.drawImage(image, 0, 0);

      // Get the date form the greyscale image (should still contains zero's)
      const scaledImageData = context.getImageData(
        0,
        0,
        this.selectedImageRef.width,
        this.selectedImageRef.height
      );

      var newimageDataCounter = 0;
      for (let yIndex = 0; yIndex < this.matrixRay.length; yIndex++) {
        const rowArray = this.matrixRay[yIndex];

        for (let xIndex = 0; xIndex < rowArray.length; xIndex++) {
          const pixelColorIndex = rowArray[xIndex];
          scaledImageData.data[newimageDataCounter] =
            pixelColorIndex.scaledGreyValue; //red
          newimageDataCounter++;
          scaledImageData.data[newimageDataCounter] =
            pixelColorIndex.scaledGreyValue; //green
          newimageDataCounter++;
          scaledImageData.data[newimageDataCounter] =
            pixelColorIndex.scaledGreyValue; //blue
          newimageDataCounter++;
          scaledImageData.data[newimageDataCounter] = 255;
          newimageDataCounter++;
        }
      }
      // Apply the eddited iamgedata
      context.putImageData(scaledImageData, 0, 0);
      this.greyImageData = scaledImageData.data;
    }
  }

  convertScaledRayToCSv(): void {
    const onlyGreyScaleRay = this.matrixRay.map(row =>
      row.map(p => p.scaledGreyValue)
    );
    const csv = convertArrayToCSV(onlyGreyScaleRay);
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
.image-previeww-rapper {
  height: 250px;
}
.image-with-overflow {
  overflow-x: auto;
  overflow-y: auto;
}

.zoom {
  transition: transform 0.2s; /* Animation */
}

.zoom:hover {
  transform: scale(
    4
  ); /* (150% zoom - Note: if the zoom is too large, it will go outside of the viewport) */
}
</style>
