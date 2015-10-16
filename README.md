# Yahoo Kimo Android Quiz

### Q1. What is a Content Provider? Please give a short example code to create, query, insert, update and delete your own content provider.

### Q2. What is Intent? Describe three common use cases for using an Intent.

### Q3. There are scenarios in which your activity is destroyed due to normal app behavior, such as when the user presses the Back button or your activity signals its own destruction by calling finish(). Please describe how to save and restore activity state, and give a example code for this.

### Q4. Which dialog ui components can you use in you Android application? Please give one dialog example code to demo how to use it.
I can use `AlertDialog`, `DatePickerDialog`, `ProgressDialog`, `TimePickerDialog` in my application. Take `AlertDialog` as an example:

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
For very simple data, I would pass them simply by appending them to the `Intent` directly. for example:

```java
public static final String KEY_ID = "custom_id";

Intent intent = new Intent(this.CurrentActivity, TargetActivity.class);
intent.putExtra(KEY_ID, "here the id is");
startActivity(intent);
```

### Q6. What is Implicit Intent? and what is explicit intent

### Q7. What is ANR? When does ANR occur?
ANR is the abbreviation of "**Application Not Responding**".

### Q8. Your photo-sharing app displays a system notification when the user receives a photo. Your app should display the photo when the user taps the notification. Which of the following do you need to attach to the Notification object that you pass to NotificationManager?  Please give a example code for this.

### Q9. What is the nine-patch Image?

### Q10. What is looper, message queue, and a Handler?
