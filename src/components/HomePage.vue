<template>
  <v-container>
    <v-row class="text-center">
      <v-col cols="12" align="center">
        <v-img class="mx-4 mt-1 logo"></v-img>
        <!-- TODO ADD LOGO -->

        <transition v-on:after-leave="animationFinished = true" name="slide">
          <v-card max-width="90vw" class="pa-1 mx-2">
            <v-card-text class="pb-0">
              <h2 class="headline font-weight-bold mb-3">
                Upload a research paper
              </h2>
              <p>
                SciSimplify will analyze it and reorganize it into an easy to
                understand format.
              </p>
            </v-card-text>
            <v-row no-gutters align="center" justify="center">
              <v-col cols="4">
                <div class="drop-zone">
                  <v-file-input
                    v-model="uploadedPDF"
                    accept=".pdf"
                    placeholder="Click to select or drag PDF file here"
                    color="#40739e"
                    class="file-input"
                    show-size
                  ></v-file-input>
                  <!-- @change="onFileChange" -->
                </div>
              </v-col>
            </v-row>
            <v-card-actions class="mx-2 mb-2">
              <v-btn
                color="#78a9ce"
                class="white--text"
                @click="analyzePDF"
                :disabled="!isPDFUploaded"
              >
                Confirm
              </v-btn>
              <v-btn
                color="#78a9ce"
                class="white--text"
                @click="resetFile"
                :disabled="!isPDFUploaded"
              >
                Reset
              </v-btn>
            </v-card-actions>

            <div v-if="sectionsContent">
              <div
                v-for="(content, index) in sectionsContent.content"
                :key="index"
              >
                <h2>{{ sectionsContent.sections[index] }}</h2>
                <p>{{ content }}</p>
              </div>
            </div>
          </v-card>
        </transition>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
export default {
  name: "HomePage",

  data: () => ({
    logo: "TODO",
    uploadedPDF: null,
    apiUrl: "local",
    awaitingServer: false,
    isPDFUploaded: false,
    isDragOver: false,
    sectionsContent: null,
  }),

  methods: {
    async analyzePDF() {
      if (this.uploadedPDF) {
        console.log("Analyzing file:", this.uploadedPDF);

        const formData = new FormData();
        formData.append("pdf", this.uploadedPDF);

        try {
          const response = await fetch("http://localhost:8000/analyze", {
            method: "POST",
            body: formData,
          });

          if (response.ok) {
            console.log("File analyzed successfully");
            const data = await response.json();
            const parsedData = JSON.parse(data.message);
            this.sectionsContent = parsedData;
            console.log(this.sectionsContent);
          } else {
            console.error("File upload failed");
          }
        } catch (error) {
          console.error("Error uploading file:", error);
        }
      }
    },

    resetFile() {
      this.uploadedPDF = null;
      this.isPDFUploaded = false;
    },
  },

  watch: {
    uploadedPDF(newValue) {
      this.isPDFUploaded = newValue !== null;
    },
  },
};
</script>


<style scoped>
.drop-zone {
  position: relative;
  border: 2px dashed #ccc;
  border-radius: 5px;
  text-align: center;
  width: 90%;
  padding: 20px;
}
.drop-zone--drag-over {
  border-color: blue;
}

.circularProgress {
  display: flex;
  position: absolute;
  z-index: 2;
  width: 100%;
  height: 100%;
  background-color: rgba(255, 255, 255, 0.5);
}

.slide-enter-active,
.slide-leave-active {
  transition: transform 0.5s;
}
.slide-enter,
.slide-leave-to {
  transform: translateY(-85%);
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.6s ease;
}
.fade-enter,
.fade-leave-to {
  opacity: 0;
}

@media only screen and (min-width: 768px) {
  .termsCard {
    max-width: 40vw !important;
  }

  .description {
    display: flex;
    align-items: center;
    width: 50%;
  }
}
</style>
