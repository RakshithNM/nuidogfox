<template>
  <main>
    <h1>{{ msg }}</h1>
    <p>Click start and give browser the permission to use microphone and say "DOG" or "FOX" to see an image of dog or fox.</p>
    <button @click="toggleStartStop">{{ recognizing ? "STOP" : "START" }}</button>
    <p>{{ currentAnimal }}</p>
    <img v-if="picture.image" :src="picture.image" alt="animal-picture">
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
    const currentAnimal = ref("")
    const recognizing = ref(false)
    const picture = ref()

    let grammar = `#JSGF V1.0; grammar animal; public <animal> = dog | fox ;`
    let recognition = new (window as any).webkitSpeechRecognition();
    let speechRecognitionList = new (window as any).webkitSpeechGrammarList();

    const reset = () => {
      recognizing.value = false;
      currentAnimal.value = "";
      picture.value = {};
    }

    const toggleStartStop = () => {
      if(recognizing.value) {
        recognition.stop();
        reset();
      }
      else {
        recognition.start();
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
    recognition.onend = recognition.start;

    recognition.onresult = async function(event: SpeechRecognitionEvent) {
      for(let i = event.resultIndex; i < event.results.length; ++i) {
        if(event.results[i].isFinal) {
          console.log(event);
          let animal = event.results[i][0].transcript;
          currentAnimal.value = animal;
          picture.value = await getAnimal(animal).catch((e) => console.log("failed to fetch animal"));
        }
      }
    }

    recognition.onstart = async function(event: SpeechRecognitionEvent) {
      console.log(event);
      console.log("started recognition");
    }

    recognition.onerror = async function(event: SpeechRecognitionEvent) {
      alert(event.type);
    }

    return { 
      currentAnimal,
      recognizing,
      toggleStartStop,
      picture
    }

  }
})
</script>

<style scoped>
p {
  color: limegreen;
}
</style>
