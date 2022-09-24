<template>
  <div
    class="relative flex items-top justify-center min-h-screen bg-gray-100 sm:items-center sm:pt-0"
  >
    <div class="w-full max-w-4xl mx-auto sm:px-6 lg:px-8 ">
      <a
        class="flex text-2xl justify-center pt-8 sm:pt-0"
        href="/"
      >
        SimilarityNet
      </a>
      <div class="mt-8 bg-white overflow-hidden shadow sm:rounded-lg p-6">
        <h2 class="text-2xl leading-7 font-semibold">
          Feature vectors
        </h2>
        <h5 class="text-gray-400">
          Shape (#Runs, #Timesteps, 128x128)
        </h5>
        <input @input="loadFileAsText" type="file" id="fileToLoad">
        <textarea
          v-model="inputString"
          @change="onInputChanged"
          placeholder="Input array"
          rows="5"
          class="form-control w-full text-gray-700 border border-solid border-gray-300 rounded transition ease-in-out focus:text-gray-700 focus:bg-white focus:border-green-600 focus:outline-none">
        </textarea>
        <div class="mt-4 pt-4 text-gray-800 border-t border-dashed">
        </div>
        <h2 class="text-2xl leading-7 font-semibold">1D temporal embedding</h2>
        <LineChart :input-csv="outputJson"></LineChart>
        <h5 class="text-gray-400">
          Values
        </h5>
        <textarea
          v-html="output"
          rows="5"
          class="form-control w-full text-gray-700 border border-solid border-gray-300 rounded transition ease-in-out focus:text-gray-700 focus:bg-white focus:border-green-600 focus:outline-none">
</textarea>

      </div>
    </div>
  </div>
</template>

<script lang="js">
import Vue from 'vue'
import * as tf from '@tensorflow/tfjs'


export default Vue.extend({
  name: 'IndexPage',

  data() {
    return {
      model: null,
      input: [],
      inputString: '',
      prediction: [],
      inputDimensionality: null,
      output: '',
      outputJson: []
    }
  },

  computed: {
    outputString() {
      const rows = [['timestep', 'value']]
      let res = 'timestep, value \n'
      for(let i = 0; i < this.prediction.length; i++) {
        rows.push([i, this.prediction[i]])
        res += `${i}, ${this.prediction[i]} \n`
      }
      return res
    },
  },

  async mounted() {
    this.model = await tf.loadGraphModel('/tfjs-model/model.json');
    this.inputDimensionality = this.model.inputs[0].shape[this.model.inputs[0].shape.length - 1]
  },

  methods: {
    onInputChanged(event) {
      this.updateInput(this.inputString)
    },
    loadFileAsText(event) {
      var fileToLoad = event.target.files[0];

      var fileReader = new FileReader();
      fileReader.onload = (fileLoadedEvent) => {
        this.inputString = fileLoadedEvent.target.result;
        console.info('File loaded')
        this.updateInput(this.inputString)
      };

      fileReader.readAsText(fileToLoad, "UTF-8");
    },
    updateInput(data) {
      try {
        const d = JSON.parse(data)

        const input = tf.tensor(d, undefined, 'float32')


        const numTimestep = input.shape[input.shape.length - 1]
        let padded = tf.pad(input, [[0, 0], [0, 0], [0, this.inputDimensionality - numTimestep]])
        if (padded.shape.length == 2) {
          padded = tf.expandDims(padded, 0)
        }


        this.input = padded
        this.prediction = this.model.predict(this.input)
        console.info(`Projections predicted.`, this.prediction)
        this.prediction = this.prediction.arraySync()
        console.log(this.prediction)

        const rows = [['run', 'timestep', 'value']]
        let outputString = 'run, timestep, value \n'
        this.outputJson = []

        this.prediction.forEach((run, i) => {
          run.forEach((timestep, t) => {
            this.outputJson.push(
              {
                timestep: t,
                value: timestep[0],
                run: i
              }
            )
          })
        })

        console.log(this.outputJson)

        return

        for(let i = 0; i < this.prediction.length; i++) {
          const ts = i % this.prediction.length
          this.outputJson.push({ timestep: ts, value: this.prediction[i]})
          rows.push([i, this.prediction[i]])
          outputString += `${i}, ${this.prediction[i]} \n`
        }
        this.output = outputString
      } catch (e) {
        console.log(e)
        // console.log(data)
        alert('Please enter a valid array.')
      }
    }
  }

})
</script>
