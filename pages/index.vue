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
        <textarea rows="5"
                  class="form-control w-full text-gray-700 border border-solid border-gray-300 rounded transition ease-in-out focus:text-gray-700 focus:bg-white focus:border-green-600 focus:outline-none">
[[[-0.97146581  0.2371743  -0.0015886  ...  0.          0.
   0.        ]
 [-0.97043235  0.241368   -0.00159416 ...  0.          0.
   0.        ]
 [-0.9694267   0.24537589 -0.00159815 ...  0.          0.
   0.        ]
 ...
 [-0.32716373  0.52453973 -0.78601652 ...  0.          0.
   0.        ]
 [-0.32412099  0.53944445 -0.77713916 ...  0.          0.
   0.        ]
 [-0.30329343  0.53769005 -0.78670357 ...  0.          0.
   0.        ]]]
        </textarea>
        <div class="mt-4 pt-4 text-gray-800 border-t border-dashed">
        </div>
        <h2 class="text-2xl leading-7 font-semibold">1D temporal embedding</h2>
        <LineChart></LineChart>
        <h5 class="text-gray-400">
          Values
        </h5>
        <textarea rows="5"
                  class="form-control w-full text-gray-700 border border-solid border-gray-300 rounded transition ease-in-out focus:text-gray-700 focus:bg-white focus:border-green-600 focus:outline-none">
timestep,value
0.000000,-0.575740
1.000000,-0.595488
2.000000,-0.614489
3.000000,-0.633121
4.000000,-0.651068
5.000000,-0.668693
6.000000,-0.688557</textarea>

      </div>
    </div>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'

export default Vue.extend({
  name: 'IndexPage',

})
</script>
