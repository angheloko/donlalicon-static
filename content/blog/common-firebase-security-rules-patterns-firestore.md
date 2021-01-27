---
title: "Common Firebase Security Rules Patterns for Cloud Firestore"
subtitle: "Common Firebase security rules that you can use on your next Firestore-based project"
lead: "Common Firebase security rules that you can use on your next Firestore-based project"
description: "Common Firebase security rules that you can use on your next Firestore-based project"
createdAt: 2020-01-20T01:44:20.197Z
updatedAt: 2020-01-22T08:54:06.990Z
cover: 
  image: "https://firebasestorage.googleapis.com/v0/b/donlalicon.appspot.com/o/common-firebase-security-rules-patterns-firestore%2Fbill-oxford-OXGhu60NwxU-unsplash.jpg?alt=media&token=aa6a1232-80a4-41db-91f5-58e22d327458"
  alt: "Common Firebase Security Rules Patterns for Cloud Firestore cover image"
  caption: "Photo by Bill Oxford on Unsplash"
  thumb: "https://firebasestorage.googleapis.com/v0/b/donlalicon.appspot.com/o/common-firebase-security-rules-patterns-firestore%2Fbill-oxford-OXGhu60NwxU-unsplash_200x200.jpg?alt=media&token=87b1ebfb-3780-4deb-9a9d-e2a9f17cd90d"
tags: 
  - Firestore
  - Firebase Rules
---
<p><a href="https://firebase.google.com/docs/rules" rel="noopener noreferrer nofollow">Firebase Security Rules</a> is a robust, flexible, and easy way to secure your data in <a href="https://firebase.google.com/docs/firestore" rel="noopener noreferrer nofollow">Cloud Firestore</a>. Here are some common patterns that you can use for your next project:</p><h2>Authenticated user access</h2><p>If you're using <a href="https://firebase.google.com/docs/auth" rel="noopener noreferrer nofollow">Firebase Authentication</a> in your application, you can use this pattern to determine access to a document depending on the current user.</p><p><strong>Example: Allow </strong><code>read</code><strong> access to authenticated users to any article.</strong></p><pre><code>service cloud.firestore {
  match /databases/{database}/documents {
    match /articles/{articleId} {
      allow read: if request.auth.uid != null;
    }
  }
}</code></pre><p>Here we access the <a href="https://firebase.google.com/docs/reference/rules/rules.firestore.Request#auth" rel="noopener noreferrer nofollow"><strong>auth</strong></a> property of the <a href="https://firebase.google.com/docs/reference/rules/rules.firestore.Request" rel="noopener noreferrer nofollow"><strong>request</strong></a> object. The request context contains other data such as the current user's authentication state, request method, and path among other things, which you can also use in your rules as we'll see later.</p><h2>Document status access</h2><p>This pattern restricts access to documents base on one or more of its fields.</p><p><strong>Example: Allow </strong><code>read</code><strong> access to published articles only.</strong></p><pre><code>service cloud.firestore {
  match /databases/{database}/documents {
    match /articles/{articleId} {
      allow read: if resource.data.published == true;
    }
  }
}</code></pre><p><strong>Example: Allow </strong><code>read</code><strong> access to products that are both active and in stock.</strong></p><pre><code>service cloud.firestore {
  match /databases/{database}/documents {
    match /products/{productId} {
      allow read: if resource.data.status == true &amp;&amp; resource.data.stock &gt; 0;
    }
  }
}</code></pre><p>This pattern extensively relies on the <a href="https://firebase.google.com/docs/reference/rules/rules.firestore.Resource.html#data" rel="noopener noreferrer nofollow"><strong>data</strong></a> property of the <a href="https://firebase.google.com/docs/reference/rules/rules.firestore.Resource" rel="noopener noreferrer nofollow"><strong>resource</strong></a> object. The resource object is essentially the document being read or written.</p><h2>Document ownership</h2><p>This pattern checks the current user's ID against a document's field.</p><p><strong>Example: Allow authors to update or delete their own articles.</strong></p><pre><code>service cloud.firestore {
  match /databases/{database}/documents {
    match /articles/{articleId} {
      allow update, delete: if request.auth.uid == resource.data.authorId
    }
  }
}</code></pre><h2>Parent access</h2><p>Cloud Firestore lets you create <a href="https://firebase.google.com/docs/firestore/data-model#subcollections" rel="noopener noreferrer nofollow">subcollections</a> and in some cases you will want to check the parent document in your rules.</p><p><strong>Example: Allow </strong><code>write</code><strong> access to a forum only if it's not locked.</strong></p><p>In this example, we have a <code>forums</code> collection and each forum document has a <code>posts</code> subcollection. You can get a reference the parent document by using the <a href="https://firebase.google.com/docs/firestore/security/rules-conditions#access_other_documents" rel="noopener noreferrer nofollow"><strong>get()</strong></a> function.</p><pre><code>service cloud.firestore {
  match /databases/{database}/documents {
    match /forums/{forumId} {
      match /posts/{postId} {
        allow create: if request.auth.uid != null &amp;&amp; get(/databases/$(database)/documents/forums/$(forumId)).data.locked == false
      }
    }
  }
}</code></pre><h2>Protected fields</h2><p>You can also protect fields from being set incorrectly with rules.</p><p><strong>Example: Ensure that the current user ID is used as the same as the author ID of an article.</strong></p><pre><code>service cloud.firestore {
  match /databases/{database}/documents {
    match /articles/{articleId} {
      // Check that the authorId is the same as the current user's ID.
      allow create, update: if request.auth.uid == request.resource.data.authorId;
    }
  }
}</code></pre><p>The rule above prevents the author ID from being incorrectly set to someone else's ID.</p><p><strong>Example: Validate field.</strong></p><pre><code>service cloud.firestore {
  match /databases/{database}/documents {
    match /candidates/{candidateId} {
      allow create: if request.resource.data.votes == 0
    }
  }
}</code></pre><p>This example will only allow new candidates to be created if the votes field is 0.</p><h2>User data</h2><p>This pattern assumes that you have a collection that stores user data.</p><p><strong>Example: Check if the current user is an admin.</strong></p><pre><code>service cloud.firestore {
  match /databases/{database}/documents {
    allow read, write: if get(/databases/$(database)/documents/users/$(request.auth.uid)).data.admin == true;
  }
}</code></pre><h2>User token</h2><p>This pattern requires you to implement <a href="https://firebase.google.com/docs/auth/admin/custom-claims" rel="noopener noreferrer nofollow">custom claims</a>. There are a <a href="https://firebase.google.com/docs/auth/admin/custom-claims?authuser=1#examples_and_use_cases" rel="noopener noreferrer nofollow">number of ways</a> you can set custom user claims but all them require you to use the <a href="https://firebase.google.com/docs/reference/admin/node/admin.auth.Auth.html#setcustomuserclaims" rel="noopener noreferrer nofollow">Firebase Admin SDK</a>.</p><p>Custom claims can be retrieved from the <a href="https://firebase.google.com/docs/reference/rules/rules.firestore.Request#auth" rel="noopener noreferrer nofollow">request authentication context</a>.</p><p><strong>Example: Check if the current user has the admin token.</strong></p><pre><code>service cloud.firestore {
  match /databases/{database}/documents {
    allow read, write: if request.auth.token.admin == true;
  }
}</code></pre>