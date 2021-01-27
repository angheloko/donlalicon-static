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
<p>You may have noticed that my posts normally will have a cover image. Originally, I did things manually where I would upload the images using the <a href="https://console.firebase.google.com/" rel="noopener noreferrer nofollow">Firebase Console</a>, obtain the download URL, and use it on the blog. It wasn't complicated but it sure can be improved, so I recently added a way to upload the cover images to Cloud Storage directly from a form.</p><p><img src="https://firebasestorage.googleapis.com/v0/b/donlalicon.appspot.com/o/images%2Fblog-form-image-upload.svg?alt=media&amp;token=dfdae6d6-c17b-41c4-9d0f-7875f66b4274"><em>(My new blog form with </em><a href="https://tiptap.scrumpy.io/" rel="noopener noreferrer nofollow"><em>tiptap</em></a><em> and an image file input element)</em></p><h2>File element</h2><p>The file element is one of those things that's a bit troublesome to style and in most cases you'd find it more practical to just hide it completely, which is what I did for my blog. The technique is common and is simply hiding the file input element and having another element trigger a <code>click</code> event on the file input element.</p><p>The snippet below demonstrates the use of this technique. In addition, a preview of the image and a delete button is added to create this "compound" element, which greatly enhances an otherwise plain file input element.</p><p>I'm using <a href="https://tailwindcss.com/" rel="noopener noreferrer nofollow">Tailwind CSS</a>. You can refer to its <a href="https://tailwindcss.com/docs/utility-first" rel="noopener noreferrer nofollow">documentation</a> for the effects of the CSS classes used.</p><pre><code>&lt;template&gt;
  &lt;div class="mb-4"&gt;
    &lt;label for="imageUrl"&gt;Image&lt;/label&gt;
    &lt;div v-if="blog.imageUrl"&gt;
      &lt;!-- A preview of the image. --&gt;
      &lt;img :src="blog.imageUrl" class="w-24 md:w-32 h-auto object-cover inline-block" alt=""&gt;
      &lt;!-- Delete button for deleting the image. --&gt;
      &lt;button
        v-if="blog.imageUrl"
        @click="deleteImage"
        :disabled="isDeletingImage"
        type="button"
        class="bg-red-500 border-red-300 text-white"
      &gt;
        {{ isDeletingImage ? 'Deleting...' : 'Delete' }}
      &lt;/button&gt;
    &lt;/div&gt;
    &lt;!-- Clicking this button triggers the "click" event of the file input. --&gt;
    &lt;button
      v-if="!blog.imageUrl"
      @click="launchImageFile"
      :disabled="isUploadingImage"
      type="button"
    &gt;
      {{ isUploadingImage ? 'Uploading...' : 'Upload' }}
    &lt;/button&gt;
    &lt;!-- This is the real file input element. --&gt;
    &lt;input
      ref="imageFile"
      @change.prevent="uploadImageFile($event.target.files)"
      type="file"
      accept="image/png, image/jpeg"
      class="hidden"
    &gt;
  &lt;/div&gt;
&lt;/template&gt;</code></pre><h2>The script</h2><p>The accompaniment to the template above, the Javascript below contains the various event handlers and demonstrates how the file is actually uploaded to the Firebase Cloud Storage, which is pretty much based from their <a href="https://firebase.google.com/docs/storage/web/upload-files" rel="noopener noreferrer nofollow">excellent documentation</a>.</p><p>A few things that might need some clarification are the use of <code>$refs</code> and <code>$firebase</code>.</p><p><a href="https://vuejs.org/v2/api/#vm-refs" rel="noopener noreferrer nofollow"><strong>$refs</strong></a> is basically any DOM element or component registered with the <code>ref</code> attribute. You may have noticed in the template our file input element with the attribute <code>ref="imageFile"</code>. This allows us to easily access the element just by using the <code>$refs</code> property. Without the attribute, the alternative would have to use <code>querySelector</code> or <code>querySelectorAll</code> in order access the file input element.</p><p>The <strong>$firebase</strong> is the <a href="https://firebase.google.com/docs/web/setup" rel="noopener noreferrer nofollow">Firebase object</a> that we've injected with the use of <a href="https://nuxtjs.org/guide/plugins/" rel="noopener noreferrer nofollow">plugins</a>. You can know more about how this was done by reading my previous post about <a href="https://donlalicon.dev/blog/connecting-universal-nuxtjs-firebase" rel="noopener noreferrer nofollow">connecting a Nuxt application with Firebase</a>.</p><pre><code>export default {
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

      const uploadTask = imageRef.put(file, metadata).then((snapshot) =&gt; {
        // Once the image is uploaded, obtain the download URL, which
        // is the publicly accessible URL of the image.
        return snapshot.ref.getDownloadURL().then((url) =&gt; {
          return url
        })
      }).catch((error) =&gt; {
        console.error('Error uploading image', error)
      })

      // When the upload ends, set the value of the blog image URL
      // and signal that uploading is done.
      uploadTask.then((url) =&gt; {
        this.blog.imageUrl = url
        this.isUploadingImage = false
      })
    },
    deleteImage () {
      this.$firebase.storage().refFromURL(this.blog.imageUrl).delete()
        .then(() =&gt; {
          this.blog.imageUrl = ''
        })
        .catch((error) =&gt; {
          console.error('Error deleting image', error)
        })
    }
  }
}
&lt;/script&gt;</code></pre><p>You can see how I used the <a href="https://github.com/angheloko/donlalicon/blob/master/components/BlogForm.vue" rel="noopener noreferrer nofollow">above snippets</a> for this blog along with its full source code from the <a href="https://github.com/angheloko/donlalicon" rel="noopener noreferrer nofollow">repo</a>.</p>