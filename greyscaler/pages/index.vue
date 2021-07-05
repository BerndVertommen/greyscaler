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
            <b-form-group
              id="selectFileFG"
              label="Selecteer een afbeelding"
              label-for="fileSelectInput"
            >
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
              :src="selectedImagePath"
              fluid
              alt="Geen bestand geselecteerd"
              ref="selectedImageRef"
            ></b-img>
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
            <b-img ref="greyScaledImage"></b-img>
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

@Component({
  components: {
    ScaleChart
  }
})
export default class HomePage extends Vue {
  selectedFile: File | null = null;
  selectedImagePath: string = "";

  scaleRay: number[] = [];

  @Ref()
  selectedImageRef!: BImg;

  @Ref()
  greyScaledImage!: BImg;

  onFileSelected(file: File): void {
    // console.log(file);
    this.selectedImagePath = URL.createObjectURL(file);

    setTimeout(() => {
      console.log("Image selected");
      this.imageToGreyScale();
    }, 1000);
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
        this.fillGreyScaleRay(result.data);
        // this.greyScaledImage = new BImg();
        this.greyScaledImage.setAttribute(
          "src",
          imageFilterCore.convertImageDataToCanvasURL(result)
        );
      });
    }
  }

  fillGreyScaleRay(resultDataRay: []): void {
    const newRay = new Array<number>();
    for (let indexNewRay = 0; indexNewRay < 256; indexNewRay++) {
      newRay.push(0);
    }

    for (let index = 0; index < resultDataRay.length; index++) {
      const greyScaleValue = resultDataRay[index];
      newRay[greyScaleValue] = newRay[greyScaleValue] + 1;
    }

    this.scaleRay = newRay;
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
