# Firebase Notes

## Using Node.js 

### Setup

Install the Firebase node/npm packages.

```bash
$ sudo npm install -g firebase
$ sudo npm install -g firebase-tools
```

### Connecting to the database

In the Firebase interface, you should be able to navigate to the general settings in your project (the gear icon).  From there you should be able to find a link that says something to the effect of "Add Firebase to your web app".  Click this and copy the general connection code:

```javascript
var config = {
  apiKey: "AIzaSyDBASDSDDF23SDVAS_ZmGSfM",
  authDomain: "foo-bar42.firebaseapp.com",
  databaseURL: "https://foo-bar42.firebaseio.com",
  storageBucket: "foo-bar42.appspot.com",
  messagingSenderId: "61922992922259161"
};
firebase.initializeApp(config);

// get a reference to the database service
var database = firebase.database();
```

### Grabbing list of data

You can grab a onetime set of data from the database.  The `ref` is the location of the collection.  In this case it's the grabbing the "bookmarks" collection, accessible at this URL: `https://bookmrks-e4ff5.firebaseio.com/bookmarks.json`.

#### Using `once`

[once](https://firebase.google.com/docs/reference/js/firebase.database.Reference#once) returns a [Firebase.Promise](https://firebase.google.com/docs/reference/js/firebase.Promise).  It accepts the event type to listen for.  In this case we're listening for the "value" event, which just means we're looking for the value.  Once listens for exactly one event and then stops listening.  The success callback of the promise passes in a [DataSnapshot](https://firebase.google.com/docs/reference/js/firebase.database.DataSnapshot).  You can then get the data off of this object, with the [val](https://firebase.google.com/docs/reference/js/firebase.database.DataSnapshot#val) method.

```javascript
let bookmarksRef = database.ref('bookmarks');
let bookmarks;
bookmarksRef.once('value', function(dataSnapshot) {
  bookmarks = dataSnapshot.val();
});
```

#### Using `on`

[on](https://firebase.google.com/docs/reference/js/firebase.database.Reference#on) works just like `once` except it is a subscription that will continue to be updated everytime the event ("value" in this case) changes.  This will give your app some real-time-ness.  Since this is essentially a subscription, you will need to call the [off](https://firebase.google.com/docs/reference/js/firebase.database.Reference#off) method to cancel the subscription:

```javascript
let bookmarksRef = database.ref('bookmarks');
let bookmarks;
let onValueChange = bookmarksRef.on('value', function(dataSnapshot) {
  bookmarks = dataSnapshot.val();
});
// then cancel the subscription with
onValueChange.off()
```