<template>
  <div class="container mt-4">
    <h2>Add Movie</h2>

    <!-- Success Message -->
    <div v-if="successMessage" class="alert alert-success">
      {{ successMessage }}
    </div>

    <!-- Error Messages -->
    <div v-if="errors.length" class="alert alert-danger">
      <ul class="mb-0">
        <li v-for="(error, index) in errors" :key="index">{{ error }}</li>
      </ul>
    </div>

    <form id="movieForm" @submit.prevent="saveMovie">
      <div class="mb-3">
        <label for="title" class="form-label">Movie Title</label>
        <input
          type="text"
          name="title"
          id="title"
          class="form-control"
          placeholder="Enter movie title"
        />
      </div>

      <div class="mb-3">
        <label for="description" class="form-label">Description</label>
        <textarea
          name="description"
          id="description"
          class="form-control"
          rows="4"
          placeholder="Enter movie description"
        ></textarea>
      </div>

      <div class="mb-3">
        <label for="poster" class="form-label">Movie Poster</label>
        <input
          type="file"
          name="poster"
          id="poster"
          class="form-control"
          accept="image/*"
        />
      </div>

      <button type="submit" class="btn btn-primary">Submit</button>
    </form>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const csrf_token     = ref('')
const successMessage = ref('')
const errors         = ref([])

function getCsrfToken() {
  fetch('/api/v1/csrf-token')
    .then((res) => res.json())
    .then((data) => {
      csrf_token.value = data.csrf_token
    })
    .catch((err) => console.error(err))
}

function saveMovie() {
  successMessage.value = ''
  errors.value         = []

  const movieForm = document.getElementById('movieForm')
  const formData  = new FormData(movieForm)

  fetch('/api/v1/movies', {
    method: 'POST',
    body: formData,
    headers: {
      'X-CSRFToken': csrf_token.value
    }
  })
    .then((res) => res.json())
    .then((data) => {
      if (data.errors) {
        errors.value = data.errors
      } else {
        successMessage.value = data.message
        movieForm.reset()
        // Refresh CSRF token after successful submit
        getCsrfToken()
      }
    })
    .catch((err) => console.error(err))
}

onMounted(() => {
  getCsrfToken()
})
</script>