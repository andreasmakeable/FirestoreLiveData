# FirestoreLiveData

Transforms Firebase Cloud Firestore queries, collections, and documents into `LiveData`.

## Getting Started

To add FirestoreLiveData to your app, add the following dependency to your module's `build.gradle` file:
```gradle
implementation "com.github.ptrbrynt:FirestoreLiveData:$latestVersion"
```

## Creating Models

The first step in transforming your Firestore queries is to create a model class. All model classes must extend `FirestoreModel` in order to work with this library:
```kotlin
data class Book(
  var title: String? = null,
  var author: String? = null,
  var timestamp: Timestamp? = null
): FirestoreModel()
```

It's also important **not** to include an 'id' property in your models, as this is included in `FirestoreModel`.

## Transforming queries, collections, and document references

You can transform any instance of `Query`, `CollectionReference`, or `DocumentReference` into a `LiveData` like so:

```kotlin
val db = FirebaseFirestore.getInstance()

val myQuery = db.collection("books").orderBy("title").asLiveData<Book>()

val myCollection = db.collection("books").asLiveData<Book>()

val myDocument = db.collection("books").document(documentId).asLiveData<Book>()
```

## Resources

The `asLiveData<T>()` methods return instances of `LiveData<FirestoreResource<T>>` for documents, and `LiveData<FirestoreResource<List<T>>>` for collections and queries.

The `FirestoreResource` class wraps the result of your query or reference and includes some additional information:

```kotlin
val myQuery = db.collection("books").orderBy("title").asLiveData<Book>()

// In your fragment or activity
myQuery.observe(this, Observer { resource: FirestoreResource<List<Book>> ->
  val status: Status = resource.status // Either SUCCESS, ERROR, or LOADING
  val data: List<Book>? = resource.data // The result of your query or reference, when status is SUCCESS
  val throwable: Throwable? = resource.throwable // If an error occurred, the details are here, when status is ERROR
  val errorMessage: String? = resource.errorMessage // A shortcut to resource.throwable?.localizedMessage
}
```

## Resource Observer

As a helpful utility, the library also includes a `ResourceObserver` class, which is an extension of `Observer` and provides easy callbacks for the three states.

You can implement a `ResourceObserver` like this:

```kotlin
myQuery.observe(this, object: ResourceObserver<List<Book>> {
  override fun onSuccess(data: List<Book>?) {
    // Handle successful result here
  }
  
  override fun onError(throwable: Throwable?, errorMessage: String?) {
    // Handle errors here
  }
  
  override fun onLoading() {
    // Handle loading state e.g. display a loading animation
  }
}
```

## Data Operations

If you have an instance of `CollectionLiveData`, you can add a new object to the collection like this:
```kotlin
val myCollection = db.collection("books").asLiveData<Book>()

myCollection.add(Book("A Book", "An Author", Timestamp.now()))

```

Similarly, you can perform the following operations on `DocumentLiveData`:

```kotlin
val myDocument = db.collection("books").document(documentId).asLiveData<Book>()

// Replaces the current data
myDocument.set(Book("Another Book", "A Superior Author", Timestamp.now())

// Update the value of a field
myDocument.update("title", "Updated Title")

// Update the values of multiple fields
myDocument.update(mapOf(Pair("title", "Updated Title"), Pair("author", "New Author")))

```

All of these operations return `Task`s, so if you need to implement callbacks you can do so.
