<template>
  <div class="min-h-screen flex flex-col justify-center items-center bg-gray-100">
    <Spinner :start="isStart" />
    <div class="flex flex-col sm:flex-row text-center justify-center items-center text-4xl md:text-5xl px-10 lg:px-0 mb-20 mt-10 sm:mt-0">
      Project About A Mask
      <img
        class="hidden sm:block mt-4 sm:mt-0 sm:ml-5 rounded-full w-40 h-40"
        src="../assets/images/mask.jpg"
        alt="mask corona virus"
      >
    </div>

    <form
      @submit.prevent="upload"
      method="POST"
      class="hero bg-white rounded-2xl shadow-lg px-6 sm:px-12 py-10 mb-4"
    >
      <div class="flex flex-col">
        <label
          for="image"
          class="mb-3 text-primary sm:text-lg tracking-wider"
        >Upload An Image</label>
        <input
          id="image"
          name="image"
          type="file"
          @keydown="resetErrors('image')"
          accept="image/png, image/jpeg"
          @change="onFileChange"
          class="text-gray-400 hover:text-black focus:text-black py-3 mb-3"
        >
        <span
          v-if="v$.image.$error"
          class="italic text-sm text-red-500 mb-4 tracking-wide"
        >{{ v$.image.$errors[0].$message }}</span>
      </div>
      <div class="flex justify-center">
        <button
          class="duration-300 flex justify-center w-1/2 hover:text-white hover:bg-primary border-primary text-primary border-2 text-lg tracking-wider uppercase rounded-lg py-2"
        >
          Send
        </button>
      </div>
    </form>
  </div>
</template>

<script>
import { reactive, ref } from 'vue';
import useVuelidate from '@vuelidate/core'
import { required, helpers } from '@vuelidate/validators'
import axiosInstance from "../helpers/axios";
import Spinner from '../components/Spinner.vue';

export default {
  name: "Home",
  components: {
    Spinner,
  },
  setup() {
    const payload = reactive({
      image: null,
    });

    const errors = ref({
      message: '',
    });

    const rules = {
      image: {
        required: helpers.withMessage('An image is required', required),
      },
    };

    const v$ = useVuelidate(rules, payload);

    let isStart = false;

    return {
      payload,
      errors,
      v$,
      isStart,
      onFileChange,
      upload,
      resetErrors,
    };

    function onFileChange(e) {
      const onFileChanges = e.target.files || e.dataTransfer.files;

      if (!onFileChanges.length){
        return;
      }

      payload.image = onFileChanges[0];
    }

    async function upload() {
      v$.value.$touch();

      if (v$.value.$invalid) {
        return;
      }

      try {
        this.isStart = true;
        const data = new FormData();
        data.append('image', payload.image);

        // @TODO the url of server that receives the image.

        const request = await axiosInstance.post('/mask', payload.image);

        console.log(request);

        // @TODO check if the request is working

        this.isStart = false;
      } catch (error) {

        // @TODO check if error.response.data is really return error from server.

        errors.value.message = error.response.data;
      }
    }

    function resetErrors(key) {
      v$.value[key].$reset();
      delete errors.value[message];
    }
  }
}
</script>

<style scoped>
.hero {
  width: 500px;
}

@media (max-width: 500px) {
  .hero {
    width: 90%;
  }
}
</style>
