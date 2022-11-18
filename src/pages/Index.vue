<template>
  <q-page class="">
    <div style="max-width: 1200px">
      <q-form @submit="onSubmit" class="q-gutter-md">
        <q-file
          name="images"
          v-model="images"
          label="Pick files"
          filled
          multiple
        />
        <div class="q-pa-md" style="max-width: 300px">
          <div class="q-gutter-md">
            <q-select name="model_type" v-model="model_type" :options="options" label="Model Type" />
          </div>
        </div>
        <div>
          <q-btn
            :loading="progress.loading"
            :percentage="progress.percentage"
            color="primary"
            type="submit"
            style="width: 150px"
          >
            Classify
            <template v-slot:loading>
              <q-spinner-gears class="on-left" />
                Classifying
            </template>
          </q-btn>
        </div>
      </q-form>
      <div class="q-gutter-md row items-start">
        <template v-for="(item, index) in results" :key="index">
          <div class="col-6">
            <q-img :src="item.url" style="max-width: 300px; height: 150px;">
              <div class="absolute-bottom text-subtitle1 text-center">
                {{ item.label }}, {{ item.inference_time }} ms
              </div>
            </q-img>
          </div>
        </template>
      </div>
    </div>
  </q-page>
</template>

<script>
import { ref } from 'vue'
import { useQuasar } from 'quasar'
import { api } from 'boot/axios'

const API_URL = 'http://localhost:5001/'

const $q = useQuasar()

export default {
  name: 'PageIndex',
  setup () {
    const progress = ref({ loading: false, percentage: 0 })
    const options = [
      {
        label: "32-bit Float",
        value: 32
      },
      {
        label: "8-bit Integer",
        value: 8
      },
      {
        label: "6-bit Integer",
        value: 6
      },
      {
        label: "4-bit Integer",
        value: 4
      },
    ]
    return {
      progress,
      model_type: ref({
        label: "32-bit Float",
        value: 32
      }),
      options,
      images: ref(null),
      results: ref(null),
      result: ref(null)
    }
  },
  methods: {
    onSubmit (evt) {
      this.results = []
      this.progress.loading = true
      this.progress.percentage = 0
      
      const formData = new FormData(evt.target)
      
      this.progress.percentage = 40

      if (this.images && this.images[0]) {
        for(const [index, image] of this.images.entries()) { 
          var result = {
            url: null
          }
          this.results.push(result)
          const reader = new FileReader()
          reader.onload = e => {
            this.results[index].url = e.target.result  
          }
          reader.readAsDataURL(image)
          this.$emit("input", image)
        }
      }

      api.post(API_URL + 'classify', formData)
        .then(response => {
          this.progress.percentage = 90

          this.$q.notify({
            position: 'top-right',
            color: 'green-4',
            textColor: 'white',
            icon: 'cloud_done',
            message: response.data.message
          })
          
          for(const [index, item] of response.data.entries()) {
            this.results[index].label = item.label
            this.results[index].confidence = item.confidence
            this.results[index].inference_time = item.inference_time
          }
          this.progress.percentage = 100
          this.progress.loading = false
          
        })
        .catch(error => {
          this.$q.notify({
            position: 'top-right',
            color: 'red-5',
            textColor: 'white',
            icon: 'warning',
            message: error.response.data.message
          })
          this.progress.percentage = 100
          this.progress.loading = false
        })
    }
  }
}
</script>
