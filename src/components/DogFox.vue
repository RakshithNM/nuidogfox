<template>
  <main v-if="isSupported">
    <h1>{{ msg }}</h1>
    <p><strong>Click start and give browser the permission to use microphone and
    say "DOG" or "FOX" to see an image of dog or fox.</strong></p>
    <button v-if="!recognizing" @click="start">START</button>
    <p><strong v-html="diagnostic"></strong></p>
    <img v-if="picture.image" :src="picture.image" alt="animal-picture">
  </main>
  <main v-else class="full">
    <p><strong>Speech Recognition feature is not supported by this browser/device, open this webiste on latest chrome and on a computer.</strong></p>
  </main>
</template>

<script lang="ts">
import { ref, defineComponent } from 'vue'

export default defineComponent({
  name: 'DogFox',
  props: {
    msg: {
      type: String,
      required: true
    }
  },
  setup: () => {
    const diagnostic = ref("")
    const recognizing = ref(false)
    const picture = ref()
    const isSupported = ref(false);
    let start;
    const isChrome = /Chrome/.test(navigator.userAgent) && /Google Inc/.test(navigator.vendor);
    const isEdge = /Edg/.test(navigator.userAgent) || /Edge/.test(navigator.userAgent);
    const isMobile = /Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini|Mobile|mobile|CriOS/i.test(navigator.userAgent)

    if((window as any).Modernizr.speechrecognition && isChrome && !isEdge && !isMobile) {
      isSupported.value = true;
      let grammar = `#JSGF V1.0; grammar animal; public <animal> = dog | fox ;`
      let recognition = new (window as any).webkitSpeechRecognition();
      let speechRecognitionList = new (window as any).webkitSpeechGrammarList();

      const reset = () => {
        recognizing.value = false;
        diagnostic.value = "";
        picture.value = {};
      }

      start = () => {
        if(recognizing.value == false) {
          recognition.start();
          diagnostic.value = `Say DOG or FOX to fetch an image`;
          recognizing.value = true;
        }
      }

      interface DogApiResponse {
        message: string
        status: string
      }

      interface FoxApiResponse {
        image: string
      }

      interface Animal {
        status: string
        image: string | null
      }

      type RandomAnimalFactory = () => Promise<Animal>

      const dogFactory: RandomAnimalFactory = async () => {
        const dogApiUrl = `https://dog.ceo/api/breeds/image/random`;
        const result: DogApiResponse = await fetch(dogApiUrl).then(data => data.json());
        if(result.status !== 'success') {
          throw new Error('dog api failure');
        }
        return { image: result.message, status: "success" }
      }

      const foxFactory: RandomAnimalFactory = async () => {
        const foxApiUrl = `https://randomfox.ca/floof/`;
        const result: FoxApiResponse = await fetch(foxApiUrl).then(data => data.json());
        return { image: result.image, status: "success" }
      }

      type NamedAnimalFactory = (name: string) => Promise<Animal>

      const createRandomNamedAnimalFactory = (randomFactories: {[key: string]: RandomAnimalFactory}): NamedAnimalFactory => {
        return async (name) => {
          const factory = randomFactories[name.trim().toLowerCase()];
          if(!factory) {
            return { image: null, status: "failure"};
          }

          try {
            return await factory();
          } catch(e) {
            console.log(e);
            return { image: null, status: "failure" }
          }
        }
      }

      const getAnimal: NamedAnimalFactory = createRandomNamedAnimalFactory({
        dog: dogFactory,
        fox: foxFactory
      })

      speechRecognitionList.addFromString(grammar, 1);
      recognition.grammars = speechRecognitionList;
      recognition.continuous = true;
      recognition.lang = 'en-US';
      recognition.interimResults = true;
      recognition.maxAlternatives = 1;
      reset();

      recognition.onresult = async function(event: SpeechRecognitionEvent) {
        console.log(event);
        for(let i = event.resultIndex; i < event.results.length; ++i) {
          if(event.results[i].isFinal) {
            let animal = event.results[i][0].transcript.trim();
            if(animal.split(" ").length > 1) {
              animal = animal.split(" ")[0];
            }
            const possibleAnimals = ['dog', 'fox'];
            if(possibleAnimals.includes(animal.toLowerCase())) {
              diagnostic.value = animal;
            } else {
              diagnostic.value = `you said <mark>${animal}</mark> or browser has no
              confidence(this can happen if you use a bluetooth mic) that you
              said the one of the right words(dog or fox)`;
            }
            picture.value = await getAnimal(animal).catch((e) => console.log("failed to fetch animal"));
          }
        }
      }

      recognition.addEventListener('end', (e: SpeechRecognitionEvent) => {
        setTimeout(() => {
          diagnostic.value = "";
          recognizing.value = true;
        }, 2000)
        diagnostic.value = `Say DOG or FOX, so we can fetch an image of a dog or
        fox`;
        recognition.start();
      });

      recognition.onstart = async function(event: SpeechRecognitionEvent) {
        console.log(event);
        console.log("started recognition");
      }

      recognition.onerror = async function(event: SpeechRecognitionErrorEvent) {
        if(event.type === "error" && event.error === "language-not-supported") {
          diagnostic.value = `Speech Recognition is not supported by the browser
          at the moment, kindly swich to latest chrome on a computer`;
        }
        if(event.type === "error" && event.error === "no-speech") {
          diagnostic.value = `You are not speaking`;
        }
      }

    } else {
      isSupported.value = false;
    }

    return {
      diagnostic,
      recognizing,
      start,
      picture,
      isSupported
    }
  }
})
</script>

<style scoped>
p {
  color: limegreen;
}
</style>
