# Android Quiz

### Q1. What is a Content Provider? Please give a short example code to create, query, insert, update and delete your own content provider.
Generally, an application can only access the data created by its own in Android for security concern. Apps cannot share data directly by using a shared memory in Android. Android provides the mechanism `Content Provider` to allow app sharing its private data for other apps to access. `Content Provider` provides a standard interface that connects data in one process with code running in another process.

Here is the link of the sample code of my custom `WeatherContentProvider` in one of my self-training project *Sunshine* created ~1 year ago on Github: [https://goo.gl/H6iJJW](https://goo.gl/H6iJJW). I have updated the Gradle version and made some modification according to the spec of API. You are welcome to clone and run it on your own device.

### Q2. What is Intent? Describe three common use cases for using an Intent.
An `Intent` is a messaging object that we could use to request an action from another app component. Usually we use `Intent` to:

1. Start an `Activity`:
    ```java
    Intent intent = new Intent(CurrentActivity.this, TargetActivity.class);
    startActivity(intent);
    ```

2. Start a `Service`:
    ```java
    Intent intent = new Intent(CurrentActivity.this, LargeFileUploadService.class);
    startService(intent);
    ```

3. Deliver a `Boradcast`:
    ```java
    Intent intent = new Intent("com.cloudchen.app.TEST_BROADCAST");
    sendBroadvast(intent);
    ```

### Q3. There are scenarios in which your activity is destroyed due to normal app behavior, such as when the user presses the Back button or your activity signals its own destruction by calling finish(). Please describe how to save and restore activity state, and give an example code for this.

Say our app has a `SignUpActivity` to handle the process of member registration and the whole registration process has been split into 3 steps. Each step has been implemented by its corresponding `Fragment` class. When user finished the registration requirement of one step and went to the next step, we increased the `currentStep` member variable by `1`. We save the value of `currentStep` in method `onSaveInstanceState(Bundle bundle)` and restore it in the method `onCreate(Bundle savedInstanceState)`. 

Here we have another choice that we could restore the value of `currentStep` in the method `onRestoreInstanceState(Bundle savedInstanceState)`. Also, we could save the user's input of each step in `onSaveInstanceState` if we decide to use all these information as parameters of an API call at the final step of registration. By doing that, we help the user keep values they input so that they have no need to input the same information again if the activity accidently destroyed because we could restore the values next time we restore this activity. However, what values (states) to save and when to restore (`onCreate()` or `onRestoreInstanceState()`) are depends on the design of the application.

```java
public class SingUpActivity extends AppCompatActivity {
    
    private static final String KEY_CURRENT_STEP = "current_step";

    int currentStep;

    FragmentTransaction fragmentTransaction;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_sign_up);
        restoreCurrentStep(savedInstanceState);
        fragmentTransaction = getSupportFragmentManager().beginTransaction();
    }

    private void restoreCurrentStep(Bundle savedInstanceState) {
        if (savedInstanceState.containsKey(KEY_CURRENT_STEP)) {
            currentStep = savedInstanceState.getInt(KEY_CURRENT_STEP, 0)
        }
    }

    @Override
    protected void onStart() {
        super.onStart();
        fragmentTransaction.replace(R.id.fragment_container, getRegStepFragment(currentStep));
        fragmentTransaction.commit();
    }

    private Fragment getRegStepFragment(int step) {
        switch (step) {
            case 1:
                return RegStep2Fragment.getInstance(this);
            case 2:
                return RegStep3Fragment.getInstance(this);
            default:
                return RegStep1Fragment.getInstance(this);
        }
    }

    @Override
    protected void onSaveInstanceState(Bundle outState) {
        super.onSaveInstanceState(outState);
        outState.putInt(KEY_CURRENT_STEP, currentStep);
    }

    @Override
    protected void onRestoreInstanceState(Bundle savedInstanceState) {
        super.onRestoreInstanceState(savedInstanceState);
        // we could also restore the currentStep variable here.
        // restoreCurrentStep(savedInstanceState);
    }
}
```

### Q4. Which dialog ui components can you use in you Android application? Please give one dialog example code to demo how to use it.
We can use `AlertDialog`, `DatePickerDialog`, `ProgressDialog`, `TimePickerDialog` in our application. Take `AlertDialog` as an example:

```java
AlertDialog alertDialog = new AlertDialog.Builder(this)
    .setMessage(R.string.no_network)
    // here we use lambda expression to replace the ugly annonymous inner class
    .setPositiveButton(android.R.string.ok, ((dialog, which) -> dialog.dismiss()))
    .setNegativeButton(android.R.string.cancel, ((dialog, which) -> dialog.dismiss()))
    .create();
alertDialog.show();
```

### Q5. How will you pass data to sub-activities? Please give a example code for this. 
For simple data, I would pass data simply by appending them to the instance of `Intent` directly. Simple means we only have a few number of data to pass and the type of data is supported by the `putExtra` method of `Intent`.for example:

```java
public static final String KEY_ID = "custom_id";
public static final String KEY_PRICE = "custom_price";

Intent intent = new Intent(getApplicationContext(), TargetActivity.class);
intent.putExtra(KEY_ID, "here the id is")
      .putExtra(KEY_PRICE, 1000);
startActivity(intent);
```

For much more complex data structure, I would define some custom data structure class or POJO which implement the `Parcelable` interface to model the data. For example, a `Product` class:

```java
public class Product implements Parcelable {
    private String sequenceId;
    private String name;
    private List<String> imageUrls = new ArrayList<>();

    public String getSequenceId() {
        return sequenceId;
    }

    public void setSequenceId(String sequenceId) {
        this.sequenceId = sequenceId;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public List<String> getImageUrls() {
        return imageUrls;
    }

    public void setImageUrls(List<String> imageUrls) {
        this.imageUrls = imageUrls;
    }

    @Override
    public int describeContents() {
        return 0;
    }

    @Override
    public void writeToParcel(Parcel dest, int flags) {
        dest.writeString(this.sequenceId);
        dest.writeString(this.name);
        dest.writeStringList(this.imageUrls);
    }

    public Product() {}

    private ProductItem(Parcel in) {
        this.sequenceId = in.readString();
        this.name = in.readString();
        this.imageUrls = in.createStringArrayList();
    }

    public static final Creator<Product> CREATOR = new Creator<Product>() {
        public Product createFromParcel(Parcel source) {
            return new Product(source);
        }

        public Product[] newArray(int size) {
            return new Product[size];
        }
    };
}
```

Then use the `putExtra(String name, Parcelable value)` method of `Intent` to pass it to sub-activities.

### Q6. What is Implicit Intent? and what is explicit intent
##### Explicit Intent
**Explicit intent** specifies the component such as `Activity` and `Service` to start by *the fully-qualified class name*. Usually, we start `Activity` or `Service` **explicitly** in our own app because we know the class name of them.

For example: 
```java
// in CurrentActivity
Intent intent = new Intent(CurrentActivity.this, TargetActivity.class);
startActivity(intent);
```

##### Implicit Intent
**Implicit Intent** does not specify the component by name but instead declare a general action to perform, which allows a component from another app to handle it.

For example: 
```java
// in CurrentActivity
Intent intent = new Intent(Intent.ACTION_VIEW);
intent.setData(Uri.parse("http://tw.bid.yahoo.com"));
if (intent.resolveActivity(getPackageManager()) != null) {
    startActivity(intent);
}
```

### Q7. What is ANR? When does ANR occur?
ANR is the abbreviation of "**Application Not Responding**". An ANR occurs if an application cannot respond to user input event such as key press or screen touch event in 5 seconds or a `BroadcastReceiver` hasn't finished executing in 10 seconds.

### Q8. Your photo-sharing app displays a system notification when the user receives a photo. Your app should display the photo when the user taps the notification. Which of the following do you need to attach to the Notification object that you pass to NotificationManager?  Please give an example code for this.
We should attach a `PendingIntent` instance to the `Notification` object. For example:
```java
public final static String MESSAGE = "message";
public final static String PHOTO_URI = "photo_uri";
public final static int PHOTO_VIEW_REQUEST_CODE = 101;
public final static int PHOTO_VIEW_NOTIFICATION_ID = 201;

String message = extras.getString(MESSAGE);
String uri = extras.getString(PHOTO_URI);

// Here we implement a photo viewer in out app, so call it explicitly
Intent intent = new Intent(this, PhotoViewerActivity.class);
intent.setData(Uri.parse(uri));

TaskStackBuilder stackBuilder = TaskStackBuilder.create(this);
stackBuilder.addParentStack(PhotoViewerActivity.class);
stackBuilder.addNextIntent(intent);
PendingIntent pendingIntent = stackBuilder.getPendingIntent(PHOTO_VIEW_REQUEST_CODE, PendingIntent.FLAG_UPDATE_CURRENT);

NotificationManager notificationManager = (NotificationManager) this.getSystemService(Context.NOTIFICATION_SERVICE);
NotificationCompat.Builder builder = new NotificationCompat.Builder(this);

builder.setSmallIcon(R.drawable.notification_icon_small)
    .setContentTitle(getString(R.string.app_name))
    .setContentText(message)
    .setLargeIcon(R.drawable.notification_icon_large)
    .setStyle(new NotificationCompat.BigTextStyle().bigText(message))
    .setAutoCancel(true)
    .setContentIntent(pendingIntent);

notificationManager.notify(PHOTO_VIEW_NOTIFICATION_ID, builder.build());
```

### Q9. What is the nine-patch Image?
A nine-path image is an image file with `.9` extension. For example: `dialog_bg.9.png`. The image itself is composed of the original image and **extra 1px-wide black line** around each edge of the image(rectangle). The black line at *left* and *top* define the so-called **stretchable patches**. Stretchable patches are the area where the system automatically use to expand vertically/horizontally of the image so that it adopt to the size of a widget. The black line at *right* and *bottom* define the optional content area. Content area defines and limits the space where our content such as text can be rendered.
The most straightforward use case of a nine-patch image is the background of the comic or manga style dialogue in IM applications. Since we have no idea how much characters user will input, we use a nine-patch image as the background so that Android system will calculate the final size of the dialogue widget according to the characters user input and automatically adopt the stretchable background according to the final size of the widget.

### Q10. What is looper, message queue, and a Handler?
`MessageQueue` is a low-level class which hold the list of `Message`s to be dispatched by a `Looper` within its associated thread. We can not access the thread's `MessageQueue` directly, instead we have to add `Message` through the `Handler` created on the same thread. We can take `Looper` as a `while` loop which is always `true` so that it never stop looping against its associated `MessageQueue` and dispatch `Message` object from `MessageQueue` to the `Handler` to consume it.

There are two main uses for a `Handler`: 

1. to schedule messages and runnables to be executed as some point in the future
2. to enqueue an action to be performed on a different thread than your own.

There are also two versions of methods which we could use to add messages to `MessageQueue`.

* The *post* version of methods such as `post(Runnable)`, `postDelayed(Runnable, long)` and so on let us enqueue `Runnable` objects to be called by the `MessageQueue` when they are received.
* The *send* version of methods such as `sendMessage(Message)`, `sendMessageDelayed(Message, long)` and so on let us enqueue a `Message` object which contains a bundle of data that will be processed by the `Handler`'s `handleMessage(Message)` method.

We could also control our `Message` or `Runnable` object being processed as soon as the message queue is ready to do so, or specify a delay before it gets processed or absolute time for it to be processed.

In a nutshell, `Looper`, `MessageQueue`, and `Handler` are classes design for multi-threading operation and communication in Android.
