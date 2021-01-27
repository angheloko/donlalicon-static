---
title: "Optimizing and Resizing Images Using Canvas API"
subtitle: "How to optimize and resize images before uploading them to Cloud Storage using Canvas API"
lead: "The Canvas API provides a means, among other things, to manipulate photos, which is perfect for optimizing and resizing images before uploading them to Cloud Firestore."
description: "How to optimize and resize images before uploading them to Cloud Storage using Canvas API"
createdAt: 2020-01-26T08:30:27.601Z
updatedAt: 2020-01-26T09:52:47.539Z
cover: 
  image: "https://firebasestorage.googleapis.com/v0/b/donlalicon.appspot.com/o/images%2Fapple-computer-desk-devices-326501.jpg?alt=media&token=2a8c63e4-e1d0-4ac2-bbfc-1d886d7f1f82"
  alt: "Optimizing and Resizing Images Using Canvas API cover image"
  caption: "Photo by Tranmautritam from Pexels"
  thumb: "https://firebasestorage.googleapis.com/v0/b/donlalicon.appspot.com/o/images%2Fapple-computer-desk-devices-326501_thumb.jpg?alt=media&token=b8546a13-4c9f-4bed-91f8-87f6ff8a447f"
tags: 
  - Nuxt
  - Canvas API
  - Firebase
  - Cloud Storage
---
In the [previous article](https://donlalicon.dev/blog/uploading-images-to-firebase-cloud-storage-with-nuxt), we talked about uploading files to [Firebase Cloud Storage](https://firebase.google.com/docs/storage) using the [Firebase Javascript SDK](https://firebase.google.com/docs/reference/js) in a Nuxt-based web application. In this article, we will look into how to optimize and resize an image before uploading it.

Canvas API
----------

According to [MDN](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API):

> The **Canvas API** provides a means for drawing graphics via [JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript) and the [HTML](https://developer.mozilla.org/en-US/docs/Web/HTML) `<canvas>` element. Among other things, it can be used for animation, game graphics, data visualization, photo manipulation, and real-time video processing.

Going through the documentation, the Canvas API seems to be ideal for manipulating images on the browser. Since our goal is to **optimize** and **resize** an image, the basic steps we need are:

1.  Resize the image using the [drawImage()](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/drawImage) function.
    
2.  Optimize the image using the [toBlob()](https://developer.mozilla.org/en-US/docs/Web/API/HTMLCanvasElement/toBlob) function.
    

Application
-----------

I decided to use this process for my blog. My blog uses 2 images - a cover image and a thumbnail version of the cover. Rather than having to manually resize the images, it would be ideal if all I had to do is upload a single image and let the application create the thumbnail variation that has a width of 640px. In addition, the original image should also be resized to a more reasonable size such as a width of 1280px.

We will build on top of the [previous article's codes](https://donlalicon.dev/blog/uploading-images-to-firebase-cloud-storage-with-nuxt). The template stays the same but the Javascript has been refactored so that it does the following:

1.  Create a resized optimized version of the original image to be used as the cover.
    
2.  Create a resized optimized version of the original image to be used as the thumbnail.
    
3.  Upload both versions to Firebase Cloud Firestore and store the download URLs in blog.
    

The code below has been edited for brevity but you can view the [full code](https://github.com/angheloko/donlalicon/blob/master/components/BlogForm.vue) from the repo. Be sure to check the inline comments for more information.

```js
export default {
  data () {
    return {
      blog: {},
      isUploadingImage: false,
      isDeletingImage: false,
      FULL_IMAGE: {
        maxDimension: 1280,
        quality: 0.9
      },
      THUMB_IMAGE: {
        maxDimension: 640,
        quality: 0.7
      }
    }
  },
  methods: {
    launchImageFile () {
      // Trigger the file input click event, which launches
      // the file dialog window.
      this.$refs.imageFile.click()
    },
    async uploadImageFile (files) {
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

      // Generate the cover.
      const fullImageResizePromise = new Promise((resolve, reject) => {
        this.generateVariation(file, this.FULL_IMAGE.maxDimension, this.FULL_IMAGE.quality, resolve)
      })

      // Generate the thumbnail.
      const thumbImageResizePromise = new Promise((resolve, reject) => {
        this.generateVariation(file, this.THUMB_IMAGE.maxDimension, this.THUMB_IMAGE.quality, resolve)
      })
      
      const images = await Promise.all([fullImageResizePromise, thumbImageResizePromise])

      // Upload the cover.
      const fullImageUploadPromise = this.uploadSingleImageFile(file.name, images[0], metadata)

      // Upload the thumbnail with a slight change in the filename.
      const thumbFileName = file.name.substring(0, file.name.lastIndexOf('.')) + '_thumb.' + file.name.substring(file.name.lastIndexOf('.') + 1)
      const thumbImageUploadPromise = this.uploadSingleImageFile(thumbFileName, images[1], metadata)

      this.isUploadingImage = true

      // Once both files have been uploaded, update the blog details.
      return Promise.all([fullImageUploadPromise, thumbImageUploadPromise])
        .then((results) => {
          this.blog.imageUrl = results[0]
          this.blog.teaserImageUrl = results[1]
        })
        .finally(() => {
          this.isUploadingImage = false
        })
    },
    uploadSingleImageFile (filename, blob, metadata) {
      // Create a reference to the destination where we're uploading
      // the file.
      const storage = this.$firebase.storage()
      const imageRef = storage.ref(`images/${filename}`)

      const uploadTask = imageRef.put(blob, metadata).then((snapshot) => {
        // Once the image is uploaded, obtain the download URL, which
        // is the publicly accessible URL of the image.
        return snapshot.ref.getDownloadURL().then((url) => {
          return url
        })
      }).catch((error) => {
        // eslint-disable-next-line no-console
        console.error('Error uploading image', error)
      })

      return uploadTask
    },
    generateVariation (file, maxDimension, quality, cb) {
      // Create an image element that will store our optimized image.
      const displayPicture = (url) => {
        const image = new Image()
        image.src = url
        image.onload = () => {
          const canvas = this.getScaledCanvas(image, maxDimension)
          
          // Once we have the resized image, further optimize it using
          // the toBlob() function.
          canvas.toBlob(cb, 'image/jpeg', quality)
        }
      }

      const reader = new FileReader()
      reader.onload = e => displayPicture(e.target.result)
      reader.readAsDataURL(file)
    },
    getScaledCanvas (image, maxDimension) {
      // This is where we actually resize the image using the
      // drawImage() function.
      const scaledCanvas = document.createElement('canvas')

      if (image.width > maxDimension || image.height > maxDimension) {
        if (image.width > image.height) {
          scaledCanvas.width = maxDimension
          scaledCanvas.height = (maxDimension * image.height) / image.width
        } else {
          scaledCanvas.width = (maxDimension * image.width) / image.height
          scaledCanvas.height = maxDimension
        }
      } else {
        scaledCanvas.width = image.width
        scaledCanvas.height = image.height
      }
      scaledCanvas
        .getContext('2d')
        .drawImage(
          image,
          0,
          0,
          image.width,
          image.height,
          0,
          0,
          scaledCanvas.width,
          scaledCanvas.height
        )
      return scaledCanvas
    }
  }
}
```

Alternatives
------------

If you are using Firebase, you can choose to optimize the images using [Cloud Functions](https://firebase.google.com/docs/functions). [This approach](https://github.com/firebase/functions-samples/tree/master/quickstarts/thumbnails) performs the optimization on the server automatically whenever a new image is uploaded. There's even a [prebuilt function](https://firebase.google.com/products/extensions/storage-resize-images) that you can immediately use without any coding. It is definitely worth checking out.