# Live Data :

LiveData is an observable data holder class that is also lifecycle aware.

Let's take a look at an example. You're going to have your UI, and then you're also going to have this LiveData object, which holds some data that you want to show on screen. The UI observes the LiveData object. This is like saying the UI wants to be notified of updates. 
Therefore, when the LiveData changes, the UI will get notified, and then the UI can redraw itself with the new data.

So in short, LiveData makes it easy to keep what's going on screen in sync with the data.

OK, so here's some actual code.LiveData objects will usually be kept in the ViewModel class.

#### Example
```kotlin
class UserProfileViewModel : ViewModel {
        private val _user = MutableLiveData<User>() 
        // public exposed Livedata, not mutable
        val user : LiveData<User>
             get() = _user
 }
```
        
If you're not sure what a ViewModel is, no worries. Check out the video.

Let's say you're creating an activity and ViewModel for a user profile. You'll have this user LiveData object that holds a User object.

#### Example
```kotlin
override fun onCreate(savedInstanceState : Bundle ? ) {
        userViewModel.user.observe(this,
                Observer {
                        user -> userNameTextView.Text = user?.name
                 }
}
```
        
Now, over in your activity's onCreate, you'll get that LiveData from the ViewModel class. Call observe on the LiveData.
For the first argument, you're going to pass in the UI, in this case the activity. The second argument is an "observer,"which is just a callback. Here you will call the code to update the UI. 

Now you can change the LiveData by calling either 

        setValue(newUser) // UI thread
        postValue(newUser) // BackGround thread


Use setValue if you're running on the UI thread, and then use postValue if you're running on a background thread. When either setValue or postValue is called,the LiveData notifies active observers.


If you're using Android Studio 3.1 or higher, LiveData and ViewModels work with Databinding.Usually you're going to go ahead and bind your View Model to your XML layout.

Now, after associating your ViewModel and Databinding layout, you just need to add this single line change to have your LiveData be properly observedwhen bound to the XML. You can now include references in your XML to your ViewModel and the LiveData stored with it.
If you use Databinding, you're going to no longer need to actually manually set up these observers.

So instead of creating this LiveData observer code that I showed you before, you could remove all that boilerplate. Instead, the TextView's XML references the LiveData directly.

What makes LiveData different from other observables is that it is also lifecycle aware. This means that it understands whether your UI is on screen, offscreen or destroyed.

LiveData knows about your UI state because you passed it when you call Observe.OK, so here's some benefits of LiveData's life cycle awareness.

So let's say your activity is not on screen, then your LiveData doesn't trigger useless UI updates. If the activity or UI gets destroyed,then the observation connection is cleaned up automatically by LiveData. Thus you'll never accidentally trigger an activity or fragment that is offscreen or destroyed to redraw itself. This is possible in part because of interfaces and classes in the Lifecycle library that are also used by framework classes.

These classes are--

- Lifecycle : 
which is an object that represents an Android lifecycle and what state it's in. 

- LifecycleOwner :  which is an interface for objects that have a lifecycle like AppCompatActivity or an activity fragment; 

- and finally, LifecycleObserver : which is an interface for observing a lifecycle. 

OK, so LiveData is a lifecycle observer. It abstracts away the need for you to deal directly with activity or factored lifecycle.
So those are the basics of working with LiveData and why it's useful.

I'm going to touch on a few more complex usages.

***Room*** is built to work well with LiveData. Room can return LiveData objects which are automatically notified when the database data changes and have their data loaded in a background thread.This makes it easy to have the UI update when your database updates. You can go ahead and learn more about Room in the introduction to Room video.

LiveData also provides transformations, including map, switchMap and a class called MediatorLiveData for your own custom transformations.


***Map*** lets you apply a function to the output of LiveData A and then propagate the results downstream to LiveData B.For example, you could use LiveData to take a user object and instead output a string of the user's combined first and last name.

***SwitchMap*** function transformation is a lot like map, but for mapping functions that emit LiveData instead of values.So an example here is if you have a bunch of users, perhaps stored in a Room database, you might have a lookup function for those users.Using switchMap, you'd have a LiveData for the user ID. Whenever the ID changes, your user lookup function would be called with that ID.
The result LiveData now references the newly found user LiveData.

OK, so no matter how many different times you call this look up function and get a different LiveData, your UI only needs to observe
the result LiveData once, which is the power of switchMap.

Now, if you want to go ahead and make your own custom data transformations, you should take a look at the ***MediatorLiveData*** class.

MediatorLiveData includes methods to add and remove source LiveData objects.You could then combine and propagate events from all these sources downstream.

#### MediatorLiveData Example
```kotlin
 init {
        userPosts = Transformations
            .switchMap(postId) { post ->
                if (post.isNullOrBlank()) {
                    AbsentLiveData.create()
                } else {
                    userPostViewModel.getPosts(postId.value!!)
                }
            }

        userComments = Transformations
            .switchMap(postId) { search ->
                if (search.isNullOrBlank()) {
                    AbsentLiveData.create()
                } else {
                    userCommentsRepository.getUserComments(postId.value!!)
                }
            }

        result.addSource(userComments) { value ->
            result.value = combineLatestData(userComments.value?.data, userPosts.value?.data)
        }
        result.addSource(userPosts) { value ->
            result.value = combineLatestData(userComments.value?.data, userPosts.value?.data)
        }
    }


    private fun combineLatestData(
        comments: List<Comments>?,
        posts: Posts?
    ): Resource<PostWithComments> {

        // Don't send a success until we have both results
        if (comments == null || posts == null) {
            return Resource.loading(null)
        }

        return Resource.success(PostWithComments(post = posts, comments = comments))
    }
```

Getting started with LiveData is simple, but there is a lot of potential for experimentation with this lifecycle aware observable.





