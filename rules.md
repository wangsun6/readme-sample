# Rules

Rules for Saveo Android Architecture

#### Must:

1. App Navigation and methods
2. 3rd Party Functions
3. Folder Structure
4. Documentation of components

#### Moderate:

1. File Naming
2. Design elements and properties naming
3. Class properties and fields

#### Should:

1. Testing New Module

<br/>

## Must:

**Try to focus on the order and pattern of different parts. Also understand the naming of different fuctions.**
a
<br/>

#### 1. App Navigation and methods

a. Activity and Intent :

> All Activity or Intent Must start from Activity

( If for any reason you have to start an activity from any inner component like adapter or fragment etc.. Use interface for calling function of parent activity )

b. Fragments:

> Fragments Must be handled by their parent fragment or activity

`e.g. Kotlin`

```kotlin

class ActivityA: AppCompatActivity{

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_onboard)
        startFragmentOtp()
    }

    fun startFragmentOtp(){
        val b = Bundle()
        val fragment = OnboardOtpFragment.newInstance(UiConstants.Fragment.ONBOARD_OTP, b)
        val ft = supportFragmentManager.beginTransaction()
        ft.replace(R.id.frame_content, fragment, OnboardOtpFragment::class.java.simpleName).commit()

    }

}

```

<br/>

c. Activity requestCode and fragment key:

> All Actiivty must start with specificRequest code, same aplied for fragment with specific key

`e.g. Kotlin`

```kotlin

class ActivityA: AppCompatActivity{

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_onboard)
        startFragmentOtp()
    }

    fun startFragmentOtp(){
        val b = Bundle()
        val fragment = OnboardOtpFragment.newInstance(UiConstants.Fragment.ONBOARD_OTP, b)
        val ft = supportFragmentManager.beginTransaction()
        ft.replace(R.id.frame_content, fragment, OnboardOtpFragment::class.java.simpleName).commit()
        // UiConstants.Fragment.ONBOARD_OTP key for fragment

    }

}

```

<br/>

d. Fragment newIntance:

> Fragment should be created by using static method newInstance:

`e.g. Kotlin`

```kotlin

class HomeSubscriptionFragment: Fragment{

    companion object {
        @JvmStatic
        fun newInstance(key: Int, b: Bundle?): Fragment {
            var bf: Bundle = b ?: Bundle()
            bf.putInt("fragment.key", key);
            val fragment = HomeSubscriptionFragment()
            fragment.arguments = bf
            return fragment
        }
    }
    // ...
}


```

<br/>

e. Activity or Fragment overrided methods ordering:

> All overrided methods from parent class should be at the top, according to their lifecycle order

`e.g. Kotlin`

```kotlin

class ActivityA: AppCompatActivity{

    override fun onCreate(savedInstanceState: Bundle?) {
        // ...
    }

    override fun onPostCreate(savedInstanceState: Bundle?) {
        // ...
    }

    override fun onOptionsItemSelected(item: MenuItem?): Boolean {
        // ...
    }

    override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
        // ...
    }

    override fun onBackPressed() {
        // ...
    }

    override fun onDestroy() {
        // ...
    }

}

```

<br/>

f. Other methods ordering:

> All other methods must start after the overrided function according to their type

`e.g. Kotlin`

```kotlin

class ActivityA: AppCompatActivity{

    override fun onCreate(savedInstanceState: Bundle?) {
        // ...
    }

    // ...

    override fun onDestroy() {
        // ...
    }

    /* *****************************************************
    *                    other Activity Start
    */

    fun startActivityOnboard(){
        // ...
    }

    fun startActivityPasswordless(){
        // ...
    }

    /* *****************************************************
    *                    other Fragment Start
    */

    fun startFragmentLogin(){
        // ...
    }

    fun startFragmentAccountCreate(){
        // ...
    }

    /* *****************************************************
    *                    other initial functions
    */

    fun initAdapter(){
        // ...
    }

    fun initTabs(){
        // ...
    }

    // ...

    /* *****************************************************
    *                    other functions
    */

    fun resetTabPostion(){
        // ...
    }

    fun calculateLikesCount(){
        // ...
    }

    // ...

    /* *****************************************************
    *                    3rd party functions
    */

    // ...

}

```

<br/>

#### 2. 3rd Party Functions

**These Pattern will be helpful for finding any methods related to popular third party library or functionality**

a. REST Api Method

> Must be called at the bottom of all function inside any class

> method must start with `api` term

`e.g. Kotlin`

```kotlin

class ActivityA: AppCompatActivity, AnkoLogger{

    override fun onCreate(savedInstanceState: Bundle?) {
        // ...
    }

    // ...

    /* *******************************************************************
     *                                 API
     */

    private fun apiSearchAudio(pTagIds: String){
         frame_progress.visibility = View.VISIBLE
         Postman.apiSearch()
                 .searchWithTag(
                         pTagIds
                 )
                 .subscribeOn(Schedulers.io())
                 .observeOn(AndroidSchedulers.mainThread())
                 .compose(bindToLifecycle())
                 .subscribe({ res ->
                     frame_progress.visibility = View.GONE
                     info("res: ")
                     info(res.toString())
                     //Todo onApiSuccess
                     updateAdapter(res)

                 }, { error ->
                     frame_progress.visibility = View.GONE
                     info { "error out" }
                     if(error is NullPointerException){
                         //Toast.makeText(context,"Error", Toast.LENGTH_LONG).show()
                     }
                     error.printStackTrace()
                     error { error }
                 })
     }

}

```

<br/>

b. Firebase Db Method

> Must be called at the bottom of all function and before `API methods`

> method must start with `firebase` term

`e.g. Kotlin`

```kotlin

class ActivityA: AppCompatActivity, AnkoLogger{

    override fun onCreate(savedInstanceState: Bundle?) {
        // ...
    }

    // ...

    /* *******************************************************************
     *                                 FIREBASE
     */

    private fun firebaseAudioValue(audioId: String ){

        var ref_audio = FirebaseDatabase.getInstance()
                .getReference(FirebaseApiHelper.urlAudio(audioId))

            RxFirebaseDatabase.observeSingleValueEvent(ref_audio, FdbAudio::class.java)
                .compose(bindToLifecycle())
                .map { audio: FdbAudio -> mAccountPostAdapter.addTop(audio) }
                    .subscribe()
    }

    // ...

    /* *******************************************************************
     *                                 API
     */

    // ...
}

```

<br/>

#### 3. Folder Structure

**These Pattern will be used for folder structure**

```
com.saveo.saeomedical
    -application
        -MyApplication (c)
        -ApplicationConstants (c)
    -firebase
        -fcm
            -FCMHelper(c)
            - ...
            -FCMConstants (c)

        -db
            -models
            -FirebaseDbApiRef (c)
            -FirebaseDbApiHelper (c)
            - ...
            -FirebaseDbConstants (c)

        -storage
            -downloader
            -uploader
            -FirebaseStorageApi (c)
            - ...
            -FirebaseStorageConstants (c)
    -room
        -entity
        -dao
        -AppDatabase
        -RoomConstants
    -ui
        -base
        -common
            -imageViewer
            -productDetails
            -...
        -components
            -auth
            -onboard
            - ...
        - ...
        -UiConstants (c)
    -utils
        -UtilsFile (c)
        - ...
    -widgets

```

<br/>

## Moderate:

**Try to focus on the naming and relation of different parts.**

<br/>

#### 1. File Naming

a. For components in java packages:

> All Activity, Service class of any component will have Activity , Service as their prefix.

`e.g. Java`

```java

//Here Onboard is a component
class ActivityOnbaord extends AppCompatActivity{
  // ...
}

```

<br/>

> Utils class will have Util as Prefix

`e.g. Java`

```java

//for String util
class UtilString{
  // ...
}

```

<br/>

> Any file inside a component should be prefixed according to component name. Add suffix according to component type

`e.g. Java`

```java

//Here ChooseCountry is a part of  Onboard component
//But has Type ChooseCountry
class OnboardFragmentChooseCountry extends Fragment{
  // ...

}

```

<br/>

c. For resources layout:

> all activity layout will start with prefix `activity_`

```
activity_onboard.xml
```

<br/>

> all fragment will start with component name and then "fragment" followed by it's type/objective

```

//for onboard component

onboard_fragment_phone.xml

onboard_fragment_otp.xml

```

<br/>

> all list item layout will have prefix with item\_

```

//for chat component

item_user.xml

item_medicine.xml


```

<br/>

> all common layout should be name according to their usage

```

//for empty activity

activity_view.xml


//for empty activity with toolbar

activity_view_toolbar.xml

//for activity with recyclerview and toolbar

activity_recyclerview_toolbar.xml


//for progress frame

frame_progress.xml


//any common buttons

action_next.xml

action_done.xml


etc..

```

<br/>

d. For resources drawables:

> all verctor drawable will start with prefix `dr_`

```
dr_circle.xml
```

<br/>

> all images will start with prefix `pic_`

```
pic_avatar.png
```

<br/>

> all selector will start with prefix `sc_`

```
sc_circle_grey_to_primary.xml
```

<br/>

e. Design Elements:

> Use FrameLayout or Linearlayout for any common layout design. Try to avoid relative and constrant layout.

> Use coordinator layout as parent layout in all activity.

> For making any area clickable other than button use Frame Layout for that

`e.g. xml`

```xml

<!-- // -->
    <FrameLayout
        android:id="@+id/podcast_link_fragment_btn_fl_play"
        android:clickable="true"
        android:focusable="true"
        android:minHeight="@dimen/length_36"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content">

        <LinearLayout
            android:layout_gravity="center"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:orientation="horizontal">


            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_gravity="center"
                android:layout_marginEnd="@dimen/offset_24"
                android:layout_marginStart="@dimen/offset_24"
                android:text="@string/podcast_link_paste"
                android:textColor="@color/md_white_1000" />

        </LinearLayout>
    </FrameLayout>

<!--  -->
```

<br/>

#### 2. Design elements and properties naming

a. all elements ids inside activity file should start with the `name of activity` + `activity`

```xml

<!--

file :  activity_home.xml

 -->

<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <!-- /// -->


        <com.radioit.widget.ImageViewCheckable
            android:id="@+id/home_activity_btn_podcast_fab"
            android:layout_width="@dimen/length_40"
            android:layout_height="@dimen/length_40"
            android:src="@drawable/sc_podcast"
            android:scaleType="centerInside"
            android:background="@drawable/sc_circle_primary"
            android:layout_gravity="center"
            android:elevation="@dimen/offset_8"
            tools:elevation="9dp"/>


    <!-- /// -->

</LinearLayout>
```

<br/>

b. all elements ids inside other file should start with the `filename` of file

```xml
<!--

file :  feed_item.xml

 -->

<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <!-- /// -->

             <com.radioit.widget.ImageViewCheckable
                    android:id="@+id/item_user_btn_iv_like"
                    android:layout_width="@dimen/length_32"
                    android:layout_height="@dimen/length_32"
                    android:layout_gravity="center"
                    android:background="@drawable/dr_circle_grey_500"
                    android:clickable="true"
                    android:focusable="true"
                    android:padding="@dimen/offset_2"
                    android:src="@drawable/sc_like_dislike_black"/>



    <!-- /// -->

</LinearLayout>
```

<br/>

#### 3. Class properties and fields

a. Class properties name should start with keywork : `m`

```kotlin
class HomeFragmentSubscription : FragmentBase(), AnkoLogger {

    //...

    var mSubscriptionPostList: MutableList<ModelSubscriptionPost> = arrayListOf()


    lateinit var mAdapterAudioPost: HomeSubscriptionAdapterPosts
    private lateinit var mDataSourceFactory: DefaultDataSourceFactory
    private lateinit var mExoPlayer: ExoPlayer

    //...

```

<br/>

a. static variable name should start with keywork : `s`

```java
public class ApplicationWorkadda extends Application {

    // ...

    private static ApplicationWorkadda sInstance;

    //...

```

<br/>

c. funtions parameter name should start with keywork : `p`

```kotlin
class HomeFragmentSubscription : FragmentBase(), AnkoLogger {

    //...

     override fun onAdapterPostsMusicClickPlay(pSubscroptionPost: ModelSubscriptionPost) {
        mExoPlayer.stop() //stop audio for now
        (activity as ActivityHome).startHomeMusicFragmentFromSubscriptionPost(pSubscroptionPost)
    }
    //...

```

<br/>

## Should:

<br/>

#### 1. Testing New Module

> Test any component inside TestActivityAll before adding it into main system
