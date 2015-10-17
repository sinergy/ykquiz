# Yahoo Kimo Android Quiz

### Q1. What is a Content Provider? Please give a short example code to create, query, insert, update and delete your own content provider.

### Q2. What is Intent? Describe three common use cases for using an Intent.
An `Intent` is a messaging object that we could use to request an action from another app component. Usually we use `Intent` to:

1. Start an `Activity`:
2. Start a `Service`:
3. Deliver a `Boradcast`:

### Q3. There are scenarios in which your activity is destroyed due to normal app behavior, such as when the user presses the Back button or your activity signals its own destruction by calling finish(). Please describe how to save and restore activity state, and give a example code for this.
```java
private static final String KEY_INPUT = "user_input";

EditText mEditText;

@Override
protected void onSaveInstanceState(Bundle outState) {
    super.onSaveInstanceState(outState);

}

@Override
protected void onRestoreInstanceState(Bundle savedInstanceState) {
    super.onRestoreInstanceState(savedInstanceState);
}
```

### Q4. Which dialog ui components can you use in you Android application? Please give one dialog example code to demo how to use it.
We can use `AlertDialog`, `DatePickerDialog`, `ProgressDialog`, `TimePickerDialog` in our application. Take `AlertDialog` as an example:

```java
AlertDialog alertDialog = new AlertDialog.Builder(this)
    .setMessage(R.string.no_network)
    // here we use lambda expression to replace the ugly annonymous class
    .setPositiveButton(android.R.string.ok, ((dialog, which) -> dialog.dismiss()))
    .setNegativeButton(android.R.string.cancel, ((dialog, which) -> dialog.dismiss()))
    .create();
alertDialog.show();
```

### Q5. How will you pass data to sub-activities? Please give a example code for this. 
For simple data, I would pass data simply by appending them to the instance of `Intent` directly. Simple means we only have few number of data to pass and the type of data is supported by the `putExtra` method of `Intent`.for example:

```java
public static final String KEY_ID = "custom_id";
public static final String KEY_PRICE = "custom_price";

Intent intent = new Intent(getApplicationContext(), TargetActivity.class);
intent.putExtra(KEY_ID, "here the id is")
      .putExtra(KEY_PRICE, 1000);
startActivity(intent);
```

For much more complex data structure, I would defined some custom data structure class or POJO which implement the `Parcelable` interface to model the data. for example, a `Product` class:

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
**Explicit intent** specifiy the component such as `Activity` and `Service` to start by *the fully-qualified class name*. Usually we start `Activity` or `Servcie` **explicity** in our-own app because we know the class name of them.

For example: 
```java
// in CurrentActivity
Intent intent = new Intent(CurrentActivity.this, TargetActivity.class);
startActivity(intent);
```

##### Implicit Intent
**Implicit Intent** do not specify the component by name, but instead declare a general action to perform, which allows a component fomr another app to handle it.

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

### Q8. Your photo-sharing app displays a system notification when the user receives a photo. Your app should display the photo when the user taps the notification. Which of the following do you need to attach to the Notification object that you pass to NotificationManager?  Please give a example code for this.
We should attach a `PendingIntent` instance to the `Notification` object.

### Q9. What is the nine-patch Image?
A nine-path image is an image file with `.9` extension. For example: `dialog_bg.9.png`. The image itself is composed of the original image and **extra 1px-wide black line** around each of the edge of image(rectangle).

### Q10. What is looper, message queue, and a Handler?
