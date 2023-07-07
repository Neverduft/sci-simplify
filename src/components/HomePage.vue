<template>
  <v-container>
    <v-row class="text-center">
      <v-col cols="14" align="center">
        <v-img class="mx-4 mt-1 logo"></v-img>
        <!-- TODO ADD LOGO -->

        <transition v-on:after-leave="animationFinished = true" name="slide">
          <v-card max-width="50vw" class="py-1 px-8 mx-2">
            <v-card-text class="pb-0">
              <h2 class="headline font-weight-bold mb-3">
                Upload a research paper
              </h2>
              <p>
                SciSimplify will analyze and reorganize it into an easy to
                understand format.<br />Additionally, you can view definitions
                of words by clicking on them.
              </p>
            </v-card-text>
            <v-row no-gutters align="center" justify="center">
              <v-col cols="8">
                <div class="drop-zone" @drop="dropHandler" @dragover.prevent>
                  <v-file-input
                    v-model="uploadedPDF"
                    accept=".pdf"
                    placeholder="Click to select or drag PDF file here"
                    color="#40739e"
                    class="file-input text-body-1"
                    show-size
                  ></v-file-input>
                  <!-- @change="onFileChange" -->
                </div>
              </v-col>
            </v-row>
            <v-card-actions class="mx-2 mb-2" style="justify-content: center">
              <v-btn
                color="#78a9ce"
                class="white--text"
                @click="analyzePDF"
                :disabled="!isPDFUploaded"
              >
                {{ this.action }}
              </v-btn>
              <!-- <v-btn
                color="#78a9ce"
                class="white--text"
                @click="resetFile"
                :disabled="!isPDFUploaded"
              >
                Reset
              </v-btn> -->
            </v-card-actions>

            <div v-if="hoveredWord" class="hover-box"
              :style="{ left: hoverBoxPosition.x + 'px', top: hoverBoxPosition.y + 'px' }"
              style="font-size: 12px !important; max-width: 30vw;">
              {{ hoveredWord }}
            </div>

            <div v-if="sectionsContent !== null">
              <!-- <div v-for="(content, index) in sectionsContent.sections" :key="index" class="text-left text-body-1"
                style="font-size: 14px !important;">
                <h2>{{ sectionsContent.section_titles[index] }}</h2>
                <p>{{ content }}</p>
              </div> -->
              <div v-for="(content, index) in sectionsContent.sections" :key="'section-' + index"
                class="text-left text-body-1" style="font-size: 12px !important;  font-family: Helvetica, sans-serif;">
                <h2>{{ sectionsContent.section_titles[index] }}</h2>
                <p>
                  <span
                    v-for="wordObj in splitParagraphIntoSpans(content)"
                    :key="'word-' + wordObj.id"
                    @click="clickShowHoverBox(wordObj.word, $event)"
                    @mouseleave="stopHighlighting"
                    class="hoverable-word"
                  >
                    {{ wordObj.word }}
                  </span>
                </p>
              </div>
            </div>
            <div class="main">
              <!-- Facebook Button -->
              <div class="icon fb">
                <i class="fa-brands fa-facebook-f"></i>
                <a class="Text" href="https://www.facebook.com/" target="_blank" style="text-decoration: none;"
                  ><span>Facebook</span></a
                >
              </div>

              <!-- Twitter Button add -->
              <div class="icon twt">
                <i class="fa-brands fa-twitter"></i>
                <a class="Text" href="https://twitter.com/?lang=de" target="_blank" style="text-decoration: none;"
                  ><span>Twitter</span></a
                >
              </div>

              <!-- Linkedin Button -->
              <div class="icon lnk">
                <i class="fa-brands fa-linkedin-in"></i>
                <span>Linkedin</span>
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
    hoveredWord: null,
    clickedWord: null,
    hoverBoxPosition: { x: 0, y: 0 },
    loadingDefinition: false,
    action: "SUMMARIZE",
  }),

  methods: {
    async analyzePDF() {
      if (this.uploadedPDF) {
        if (this.sectionsContent !== null) {
          this.sectionsContent = null;
          this.action = "SUMMARIZE";
          this.uploadedPDF = null;
          return;
        }

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

    async fetchWordDefinition(word) {
      this.loadingDefinition = true;
      try {
        const response = await fetch(
          `https://api.dictionaryapi.dev/api/v2/entries/en/${word}`
        );

        if (response.ok) {
          const data = await response.json();
          return data[0].meanings[0].definitions[0].definition;
        } else {
          console.error("Failed to fetch word definition");
          return null;
        }
      } catch (error) {
        console.error("Error fetching word definition:", error);
        return null;
      } finally {
        this.loadingDefinition = false;
      }
    },

    dropHandler(event) {
      // Prevent default behavior (prevent file from being opened)
      event.preventDefault();

      if (event.dataTransfer.items) {
        // Use DataTransferItemList interface to access the file(s)
        for (let i = 0; i < event.dataTransfer.items.length; i++) {
          // If dropped items aren't files, reject them
          if (event.dataTransfer.items[i].kind === "file") {
            const file = event.dataTransfer.items[i].getAsFile();
            this.uploadedPDF = file;
          }
        }
      } else {
        // Use DataTransfer interface to access the file(s)
        for (let i = 0; i < event.dataTransfer.files.length; i++) {
          this.uploadedPDF = event.dataTransfer.files[i];
        }
      }

      // Clear the drag data cache (for all formats/types)
      event.dataTransfer.clearData();
      this.isDragOver = false;
    },

    resetFile() {
      this.uploadedPDF = null;
      this.isPDFUploaded = false;
    },

    splitParagraphIntoSpans(para) {
      if(para === null){
        return;
      }
      return para.split(' ').map((word, i) => {
        return { word: word, id: i };
      });
    },

    async clickShowHoverBox(word, event) {
      if (word.length >= 3) {
        word = word.replace(",", "").replace(".", "");
        this.clickedWord = word;
        var definition = await this.fetchWordDefinition(word);
        definition = `[${word}]\n` + definition;
        this.hoveredWord = definition ? definition : "Definition not found";
        this.hoverBoxPosition.x = event.clientX;
        this.hoverBoxPosition.y = event.clientY;
      }
    },

    hideHoverBox() {
      this.hoveredWord = null;
    },

    stopHighlighting() {
      this.hoveredWord = null;
    },
  },

  watch: {
    uploadedPDF(newValue) {
      this.isPDFUploaded = newValue !== null;
    },

    sectionsContent(newValue) {
      if (newValue !== null) {
        this.action = "RESET";
      }
    },
  },
};
</script>

<style scoped>
.hover-box {
  position: fixed;
  border: 1px solid #000;
  background-color: #fff;
  padding: 10px;
  z-index: 1000;
  pointer-events: none;
}

.hoverable-word {
  cursor: pointer;
  transition: background-color 0.3s;
}

.hoverable-word:hover {
  background-color: yellow;
}

.hoverable-word {
  cursor: pointer;
}

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
.icon {
  width: 50px;
  height: 50px;
  border-radius: 100px;
  background: #fff;
  margin: 20px;
  text-align: center;
  font-size: 50px;
  line-height: 40px;
  font-family: sans-serif;
  overflow: hidden;
  box-shadow: 5px 10px 20px rgba(150, 150, 150, 0.3);
  transition: all 0.3s ease-out;
}
.icon:hover {
  width: 400px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: space-evenly;
  color: #fff;
}
.icon:hover i {
  color: #fff;
}
.icon .fa-facebook-f {
  color: #1a6ed8;
}
.fb:hover {
  background: #1a6ed8;
}

.icon .fa-twitter {
  color: #1da1f2;
}
.twt:hover {
  background: #1da1f2;
}
.icon .fa-linkedin-in {
  color: #0077b5;
}
.lnk:hover {
  background: #0077b5;
}
.icon .fa-github {
  color: #000;
}
.git:hover {
  background: #000;
}
.icon .fa-youtube {
  color: #fe0000;
}
.yt:hover {
  background: #fe0000;
}

/* responsive */
@media only screen and (min-width: 320px) and (max-width: 991px) {
  .main {
    flex-direction: column;
  }
}
.main {
  width: 100%;
  height: 100px;
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: white;
}
.Text {
  text-decoration: none !important;
}
</style>
