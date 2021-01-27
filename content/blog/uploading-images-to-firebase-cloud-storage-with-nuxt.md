---
title: "Uploading Images to Firebase Cloud Storage With Nuxt"
subtitle: "How to upload files to Firebase Cloud Storage in your Nuxt Application"
lead: ""
description: "Upload your files to Firebase Cloud Storage in Nuxt"
createdAt: 2020-01-25T08:09:32.521Z
updatedAt: 2020-01-26T08:28:05.267Z
cover: 
  image: "https://firebasestorage.googleapis.com/v0/b/donlalicon.appspot.com/o/images%2Fhector-j-rivas-QNc9tTNHRyI-unsplash.jpg?alt=media&token=f712ef7b-0df8-429b-8a6e-ecb29b3936f0"
  alt: "Uploading Images to Firebase Cloud Storage With Nuxt cover image"
  caption: "Photo by Héctor J. Rivas on Unsplash"
  thumb: "https://firebasestorage.googleapis.com/v0/b/donlalicon.appspot.com/o/images%2Fhector-j-rivas-QNc9tTNHRyI-unsplash_thumb.jpg?alt=media&token=b1a205cd-a659-456f-8783-8096079fe15c"
tags: 
  - Firebase
  - Cloud Storage
  - Nuxt
---
You may have noticed that my posts normally will have a cover image. Originally, I did things manually where I would upload the images using the [Firebase Console](https://console.firebase.google.com/), obtain the download URL, and use it on the blog. It wasn't complicated but it sure can be improved, so I recently added a way to upload the cover images to Cloud Storage directly from a form.

![](https://firebasestorage.googleapis.com/v0/b/donlalicon.appspot.com/o/images%2Fblog-form-image-upload.svg?alt=media&token=dfdae6d6-c17b-41c4-9d0f-7875f66b4274)_(My new blog form with_ [_tiptap_](https://tiptap.scrumpy.io/) _and an image file input element)_

File element
------------

The file element is one of those things that's a bit troublesome to style and in most cases you'd find it more practical to just hide it completely, which is what I did for my blog. The technique is common and is simply hiding the file input element and having another element trigger a `click` event on the file input element.

The snippet below demonstrates the use of this technique. In addition, a preview of the image and a delete button is added to create this "compound" element, which greatly enhances an otherwise plain file input element.

I'm using [Tailwind CSS](https://tailwindcss.com/). You can refer to its [documentation](https://tailwindcss.com/docs/utility-first) for the effects of the CSS classes used.

```
<template>
  <div class="mb-4">
    <label for="imageUrl">Image</label>
    <div v-if="blog.imageUrl">
      <!-- A preview of the image. -->
      <img :src="blog.imageUrl" class="w-24 md:w-32 h-auto object-cover inline-block" alt="">
      <!-- Delete button for deleting the image. -->
      <button
        v-if="blog.imageUrl"
        @click="deleteImage"
        :disabled="isDeletingImage"
        type="button"
        class="bg-red-500 border-red-300 text-white"
      >
        {{ isDeletingImage ? 'Deleting...' : 'Delete' }}
      </button>
    </div>
    <!-- Clicking this button triggers the "click" event of the file input. -->
    <button
      v-if="!blog.imageUrl"
      @click="launchImageFile"
      :disabled="isUploadingImage"
      type="button"
    >
      {{ isUploadingImage ? 'Uploading...' : 'Upload' }}
    </button>
    <!-- This is the real file input element. -->
    <input
      ref="imageFile"
      @change.prevent="uploadImageFile($event.target.files)"
      type="file"
      accept="image/png, image/jpeg"
      class="hidden"
    >
  </div>
</template>
```

The script
----------

The accompaniment to the template above, the Javascript below contains the various event handlers and demonstrates how the file is actually uploaded to the Firebase Cloud Storage, which is pretty much based from their [excellent documentation](https://firebase.google.com/docs/storage/web/upload-files).

A few things that might need some clarification are the use of `$refs` and `$firebase`.

[**$refs**](https://vuejs.org/v2/api/#vm-refs) is basically any DOM element or component registered with the `ref` attribute. You may have noticed in the template our file input element with the attribute `ref="imageFile"`. This allows us to easily access the element just by using the `$refs` property. Without the attribute, the alternative would have to use `querySelector` or `querySelectorAll` in order access the file input element.

The **$firebase** is the [Firebase object](https://firebase.google.com/docs/web/setup) that we've injected with the use of [plugins](https://nuxtjs.org/guide/plugins/). You can know more about how this was done by reading my previous post about [connecting a Nuxt application with Firebase](https://donlalicon.dev/blog/connecting-universal-nuxtjs-firebase).

```
export default {
  data () {
    return {
      blog: {},
      isUploadingImage: false,
      isDeletingImage: false
    }
  },
  methods: {
    launchImageFile () {
      // Trigger the file input click event.
      this.$refs.imageFile.click()
    },
    uploadImageFile (files) {
      if (!files.length) {
        return
      }
      const file = files[0]

      if (!file.type.match('image.*')) {
        alert('Please upload an image.')
        return
      }

      const metadata = {
        contentType: file.type
      }

      this.isUploadingImage = true

      // Create a reference to the destination where we're uploading
      // the file.
      const storage = this.$firebase.storage()
      const imageRef = storage.ref(`images/${file.name}`)

      const uploadTask = imageRef.put(file, metadata).then((snapshot) => {
        // Once the image is uploaded, obtain the download URL, which
        // is the publicly accessible URL of the image.
        return snapshot.ref.getDownloadURL().then((url) => {
          return url
        })
      }).catch((error) => {
        console.error('Error uploading image', error)
      })

      // When the upload ends, set the value of the blog image URL
      // and signal that uploading is done.
      uploadTask.then((url) => {
        this.blog.imageUrl = url
        this.isUploadingImage = false
      })
    },
    deleteImage () {
      this.$firebase.storage().refFromURL(this.blog.imageUrl).delete()
        .then(() => {
          this.blog.imageUrl = ''
        })
        .catch((error) => {
          console.error('Error deleting image', error)
        })
    }
  }
}
</script>
```

You can see how I used the [above snippets](https://github.com/angheloko/donlalicon/blob/master/components/BlogForm.vue) for this blog along with its full source code from the [repo](https://github.com/angheloko/donlalicon).