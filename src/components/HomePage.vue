<template>
  <v-container>
    <v-row class="text-center">
      <v-col
        cols="12"
        style="background-color: #40739e; box-shadow: 0 0 8px 8px #40739e"
        align="center"
      >
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
              <v-col cols="6">
                <v-file-input
                  v-model="uploadedPDF"
                  accept=".pdf"
                  placeholder="Click to select or drag PDF file here"
                  prepend-icon="mdi-file-pdf"
                  color="#40739e"
                ></v-file-input>
              </v-col>
            </v-row>
            <v-card-actions class="mx-2 mb-2">
              <v-btn
                color="#78a9ce"
                class="white--text"
                @click="processFile"
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
  }),

  methods: {
    async processFile() {
      if (this.uploadedPDF) {
        console.log("Processing file:", this.uploadedPDF);

        const formData = new FormData();
        formData.append("pdf", this.uploadedPDF);

        try {
          const response = await fetch("http://localhost:8000/upload", {
            method: "POST",
            body: formData,
          });

          if (response.ok) {
            console.log("File uploaded successfully");
            // Process the response data as needed
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
