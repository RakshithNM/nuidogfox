<template>
  <main v-if="isSupported">
    <h1>{{ msg }}</h1>
    <p><strong>Click start and give browser the permission to use microphone and
    say "DOG" or "FOX" to see an image of dog or fox.</strong></p>
    <button @click="toggleStartStop">{{ recognizing ? "STOP" : "START" }}</button>
    <p><strong>{{ diagnostic }}</strong></p>
    <img v-if="picture.image" :src="picture.image" alt="animal-picture">
  </main>
  <main v-else class="full">
    <p><strong>Speech Recognition Feature is not supported by this browser, consider updating to the latest chrome and try using it on a computer.</strong></p>
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
    let toggleStartStop;

    if((window as any).Modernizr.speechrecognition) {
      isSupported.value = true;
      let grammar = `#JSGF V1.0; grammar animal; public <animal> = dog | fox ;`
      let recognition = new (window as any).webkitSpeechRecognition();
      let speechRecognitionList = new (window as any).webkitSpeechGrammarList();

      const reset = () => {
        recognizing.value = false;
        diagnostic.value = "";
        picture.value = {};
      }

      toggleStartStop = () => {
        if(recognizing.value === true) {
          recognition.stop();
          reset();
          return;
        }
        recognition.start();
        recognizing.value = true;
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
        const fetchApiUrl = `https://randomfox.ca/floof/`;
        const result: FoxApiResponse = await fetch(fetchApiUrl).then(data => data.json());
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
        for(let i = event.resultIndex; i < event.results.length; ++i) {
          if(event.results[i].isFinal) {
            console.log(event);
            let animal = event.results[i][0].transcript.trim();
            if(animal.split(" ").length > 1) {
              animal = animal.split(" ")[0];
            }
            diagnostic.value = animal;
            picture.value = await getAnimal(animal).catch((e) => console.log("failed to fetch animal"));
          }
        }
      }

      recognition.addEventListener('end', (e: SpeechRecognitionEvent) => {
        setTimeout(() => {
          recognition.start();
          diagnostic.value = "";
          recognizing.value = true;
        }, 1000)
        diagnostic.value = `You tried stopping the recognition, starting again
        in one second`;
      });

      recognition.onstart = async function(event: SpeechRecognitionEvent) {
        console.log(event);
        console.log("started recognition");
      }

      recognition.onerror = async function(event: SpeechRecognitionErrorEvent) {
        console.error(event);
        if(event.type === "error" && event.error === "language-not-supported") {
          diagnostic.value = `Speech Recognition is not supported by the browser
          at the moment, kindly swich to latest chrome on a computer`;
        }
        console.log("an error occured");
      }

      recognition.onspeechend = function() {
        recognition.stop();
        recognizing.value = false;
        console.log("speech recognition has stopped");
      }

    } else {
      isSupported.value = false;
    }

    return {
      diagnostic,
      recognizing,
      toggleStartStop,
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
