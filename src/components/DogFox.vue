<template>
  <h1>{{ msg }}</h1>
  <p>Click start and give browser the permission to use microphone and say "DOG" or "FOX" to see an image of dog or fox.</p>
  <button @click="toggleStartStop">{{ recognizing ? "STOP" : "START" }}</button>
  <p>{{ currentAnimal }}</p>
  <img v-if="picture.image" :src="picture.image" alt="animal-picture">
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
    interface IAnimal {
      image?: string | null
      message?: string
      status: string
    }
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
        console.log("started recognition")
        recognizing.value = true;
      }
    }

    const getAnimal = async (inAnimal: string): Promise<IAnimal> => {
      const isDogWord: boolean = ["dog", "Dog", "DOG"].includes(inAnimal.trim())
      const isFoxWord: boolean = ["fox", "Fox", "FOX"].includes(inAnimal.trim())
      let result: IAnimal = { image: null, status: "tofetch" };
      if(isDogWord) {
        result = await fetch("https://dog.ceo/api/breeds/image/random").then((data) => data.json());
        if(result.status === "success") {
          return {
            image: result.message,
            status: result.status
          }
        }
      }
      if(isFoxWord) {
        result = await fetch("https://randomfox.ca/floof/").then((data) => data.json());
        return {
          image: result.image,
          status: "success"
        }
      }
      return {
        image: null,
        status: "failure"
      }
    };

    speechRecognitionList.addFromString(grammar, 1);
    recognition.grammars = speechRecognitionList;
    recognition.continuous = true;
    recognition.lang = 'en-US';
    recognition.interimResults = true;
    recognition.maxAlternatives = 1;
    reset();
    recognition.onend = reset();

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
