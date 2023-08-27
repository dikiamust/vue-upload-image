<template>
  <div>
    <h1>Upload Image</h1>
    <input
      style="display: none;"
      type="file"
      @change="onFileSelected"
      ref="fileInput"
    />

    <button type="button" class="btn btn-secondary pick" @click="$refs.fileInput.click()">Pick Image</button>
    <button type="button" class="btn btn-secondary"  @click="onUpload">Upload</button>

    <div v-if="uploadProgress !== null" class="mt-2">
      <progress :value="uploadProgress" max="100"></progress>
    </div>

    <p v-if="errorMessage" class="error-message">{{ errorMessage }}</p>
    <p v-if="successMessage" class="success-message">{{ successMessage }}</p>

    <h3 class="mt-5">Images Gallery</h3>
    <div v-if="uploadedImageUrls.length > 0">
      <div class="container-fluid">
        <div class="row">
          <div v-for="(imageUrl, index) in uploadedImageUrls" :key="index" class="col-lg-2 col-md-3 col-sm-4 col-xs-6 column">
            <img :src="imageUrl" alt="Uploaded Image">
          </div>
        </div>
      </div>
    </div>
    <p v-else>No image</p>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'UploadImage',
  data() {
    return {
      selectedFile: null,
      uploadedImageUrls: JSON.parse(localStorage.getItem('uploadedImageUrls')) || [],
      errorMessage: '',
      successMessage: '',
      uploadProgress: null,
    };
  },
  methods: {
    onFileSelected(event) {
      this.selectedFile = event.target.files[0];
    },
    onUpload() {
      this.errorMessage = '';
      this.successMessage = '';

      if (!this.selectedFile) {
        this.errorMessage = 'Please select an image.';
        return;
      }

      const allowedFormats = ['image/jpeg', 'image/png', 'image/gif', 'image/bmp', 'image/jpg'];
      if (!allowedFormats.includes(this.selectedFile.type)) {
        this.errorMessage = 'Hanya boleh file gamba';
        return;
      }

      if (this.selectedFile.size > 2 * 1024 * 1024) {
        this.errorMessage = 'File gambar terlalu besar';
        return;
      }

      const fd = new FormData();
      fd.append('image', this.selectedFile);

      const apiKey = 'your-api-key';
      const api = `https://api.imgbb.com/1/upload?expiration=600&key=${apiKey}`;
      axios
        .post(api, fd, {
          onUploadProgress: (uploadEvent) => {
            const progress = Math.round((uploadEvent.loaded / uploadEvent.total) * 100);
            this.uploadProgress = progress;

          },
        })
        .then((res) => {
          const uploadedImageUrl = res.data.data.url;

          // Append the new URL to the array
          this.uploadedImageUrls.push(uploadedImageUrl);

          // Save the updated array back to local storage
          localStorage.setItem('uploadedImageUrls', JSON.stringify(this.uploadedImageUrls));

          this.successMessage = 'Gambar berhasil diupload';
          setTimeout(() => {
            this.successMessage = '';
            this.uploadProgress = null
          }, 2000);
        })
        .catch((error) => {
          console.error('Upload error:', error);
          this.errorMessage = 'Image upload failed.';
        });
    },
  },
};
</script>

<style>

.error-message {
  color: red;
  margin-top: 10px;
}

.success-message {
  color: green;
  margin-top: 10px;
}

.row {
  display: flex;
  flex-wrap: wrap;
  padding: 0 4px;
}

/* Create four equal columns that sits next to each other */
.column {
  flex: 25%;
  max-width: 33.3%;
  padding: 0 4px;
}

.column img {
  margin-top: 8px;
  vertical-align: middle;
  width: 100%;
  filter: grayscale(1) brightness(0.5);
  border-radius: 5px;
  cursor: pointer;
  transition: 0.3s linear;
}
.column img:hover {
  filter: grayscale(0);
}
button.btn.btn-secondary.pick {
  margin-right: 10px;
}
@media screen and (max-width: 800px) {
  .column {
    flex: 50%;
    max-width: 50%;
  }
}

/* Responsive layout - makes the two columns stack on top of each other instead of next to each other */
@media screen and (max-width: 600px) {
  .column {
    flex: 100%;
    max-width: 100%;
  }
  .column img {
    filter: grayscale(0) brightness(1);
  }
}

</style>
