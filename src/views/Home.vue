<template>
  <div class="home">
    <v-container>
      <v-row>
        <v-col cols="12" md="6">
          <v-form ref="voice" lazy-validation>
            Seleccionar Voz:
            <v-select
              :loading="loading"
              :rules="rules"
              label="Voces"
              item-text="name"
              item-value="lang"
              :items="voices"
              v-model="selectedVoice"
            ></v-select>
            Seleccionar Pitch:
            <v-slider
              max="2"
              min="0"
              step="0.1"
              thumb-label
              v-model="pitch"
            ></v-slider>
            Seleccionar Rate:
            <v-slider
              max="3"
              min="0.1"
              step="0.1"
              thumb-label
              v-model="rate"
            ></v-slider>
            Seleccionar volumen:
            <v-slider
              max="1"
              min="0"
              step="0.1"
              thumb-label
              v-model="volume"
            ></v-slider>
          </v-form>
        </v-col>

        <v-col cols="12" md="6">
          <v-form ref="text" lazy-validation>
            <v-textarea
              ref="text"
              name="text"
              label="Texto a leer"
              :rules="rules"
              v-model="text"
            ></v-textarea>
            <v-btn
              block
              color="success"
              @click="toSpeak()"
              :loading="speaking"
              :disabled="speaking"
            >
              Leer
              <template v-slot:loader>
                <span>
                  <v-icon color="green darken-2" class="fas fa-volume-up">
                  </v-icon>
                </span>
              </template>
            </v-btn>
          </v-form>
          {{ userAgent }}
        </v-col>
      </v-row>
    </v-container>
  </div>
</template>

<script>
// @ is an alias to /src

export default {
  name: "Home",
  components: {},
  data: function () {
    return {
      voices: [],
      loading: true,
      speaking: false,
      rules: [(v) => !!v || "El campo no puede estar vacio"],
      text: null,
      selectedVoice: null,
      synth: window.speechSynthesis,
      pitch: 1,
      rate: 1,
      volume: 1,
      speak: new window.SpeechSynthesisUtterance(),
    };
  },
  methods: {
    async getVoices() {
      //Para firefox
      this.voices = await this.synth.getVoices();
      if (this.voices.length) {
        this.loading = false;
      }
      //Para chrome
      this.synth.onvoiceschanged = async () => {
        this.voices = await this.synth.getVoices();
        this.loading = false;
      };
    },
    toSpeak() {
      if (this.$refs.voice.validate() && this.$refs.text.validate()) {
        try {
          //texto a ser sintetizado
          this.speak.text = this.text;
          // voz que usara la aplicacion
          this.speak.voice = this.voices.find((voice) => {
            return voice.lang === this.selectedVoice;
          });
          /** Pitch
           * valor entre 0 y 2 inclusivos el valor por defecto es 1
           */
          this.speak.pitch = this.pitch;
          /** Rate
           * valor entre 0.1 y 10 inclusivos el valor por defecto es 1
           * Algunas voces no se reproducen a valores mayor a 3
           */
          this.speak.rate = this.rate;
          /** Volume
           * valor entre 0 y 1 inclusivos el valor por defecto es 1
           */
          this.speak.volume = this.volume;
          this.synth.speak(this.speak);

          //Lanzado cuando se empieza a hablar
          this.speak.onstart = () => {
            this.speaking = true;
          };
          //Lanzado cuando se termina de hablar
          this.speak.onend = () => {
            this.speaking = false;
          };
        } catch (error) {
          console.log(error);
        }
      }
    },
  },
  async mounted() {
    await this.getVoices();
  },
};
</script>
