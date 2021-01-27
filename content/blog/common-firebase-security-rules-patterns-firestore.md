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
[Firebase Security Rules](https://firebase.google.com/docs/rules) is a robust, flexible, and easy way to secure your data in [Cloud Firestore](https://firebase.google.com/docs/firestore). Here are some common patterns that you can use for your next project:

Authenticated user access
-------------------------

If you're using [Firebase Authentication](https://firebase.google.com/docs/auth) in your application, you can use this pattern to determine access to a document depending on the current user.

**Example: Allow** `read` **access to authenticated users to any article.**

```
service cloud.firestore {
  match /databases/{database}/documents {
    match /articles/{articleId} {
      allow read: if request.auth.uid != null;
    }
  }
}
```

Here we access the [**auth**](https://firebase.google.com/docs/reference/rules/rules.firestore.Request#auth) property of the [**request**](https://firebase.google.com/docs/reference/rules/rules.firestore.Request) object. The request context contains other data such as the current user's authentication state, request method, and path among other things, which you can also use in your rules as we'll see later.

Document status access
----------------------

This pattern restricts access to documents base on one or more of its fields.

**Example: Allow** `read` **access to published articles only.**

```
service cloud.firestore {
  match /databases/{database}/documents {
    match /articles/{articleId} {
      allow read: if resource.data.published == true;
    }
  }
}
```

**Example: Allow** `read` **access to products that are both active and in stock.**

```
service cloud.firestore {
  match /databases/{database}/documents {
    match /products/{productId} {
      allow read: if resource.data.status == true && resource.data.stock > 0;
    }
  }
}
```

This pattern extensively relies on the [**data**](https://firebase.google.com/docs/reference/rules/rules.firestore.Resource.html#data) property of the [**resource**](https://firebase.google.com/docs/reference/rules/rules.firestore.Resource) object. The resource object is essentially the document being read or written.

Document ownership
------------------

This pattern checks the current user's ID against a document's field.

**Example: Allow authors to update or delete their own articles.**

```
service cloud.firestore {
  match /databases/{database}/documents {
    match /articles/{articleId} {
      allow update, delete: if request.auth.uid == resource.data.authorId
    }
  }
}
```

Parent access
-------------

Cloud Firestore lets you create [subcollections](https://firebase.google.com/docs/firestore/data-model#subcollections) and in some cases you will want to check the parent document in your rules.

**Example: Allow** `write` **access to a forum only if it's not locked.**

In this example, we have a `forums` collection and each forum document has a `posts` subcollection. You can get a reference the parent document by using the [**get()**](https://firebase.google.com/docs/firestore/security/rules-conditions#access_other_documents) function.

```
service cloud.firestore {
  match /databases/{database}/documents {
    match /forums/{forumId} {
      match /posts/{postId} {
        allow create: if request.auth.uid != null && get(/databases/$(database)/documents/forums/$(forumId)).data.locked == false
      }
    }
  }
}
```

Protected fields
----------------

You can also protect fields from being set incorrectly with rules.

**Example: Ensure that the current user ID is used as the same as the author ID of an article.**

```
service cloud.firestore {
  match /databases/{database}/documents {
    match /articles/{articleId} {
      // Check that the authorId is the same as the current user's ID.
      allow create, update: if request.auth.uid == request.resource.data.authorId;
    }
  }
}
```

The rule above prevents the author ID from being incorrectly set to someone else's ID.

**Example: Validate field.**

```
service cloud.firestore {
  match /databases/{database}/documents {
    match /candidates/{candidateId} {
      allow create: if request.resource.data.votes == 0
    }
  }
}
```

This example will only allow new candidates to be created if the votes field is 0.

User data
---------

This pattern assumes that you have a collection that stores user data.

**Example: Check if the current user is an admin.**

```
service cloud.firestore {
  match /databases/{database}/documents {
    allow read, write: if get(/databases/$(database)/documents/users/$(request.auth.uid)).data.admin == true;
  }
}
```

User token
----------

This pattern requires you to implement [custom claims](https://firebase.google.com/docs/auth/admin/custom-claims). There are a [number of ways](https://firebase.google.com/docs/auth/admin/custom-claims?authuser=1#examples_and_use_cases) you can set custom user claims but all them require you to use the [Firebase Admin SDK](https://firebase.google.com/docs/reference/admin/node/admin.auth.Auth.html#setcustomuserclaims).

Custom claims can be retrieved from the [request authentication context](https://firebase.google.com/docs/reference/rules/rules.firestore.Request#auth).

**Example: Check if the current user has the admin token.**

```
service cloud.firestore {
  match /databases/{database}/documents {
    allow read, write: if request.auth.token.admin == true;
  }
}
```