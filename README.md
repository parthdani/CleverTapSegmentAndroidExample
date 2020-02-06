# CleverTap - Segment Integration in Android

**Getting Started i.e. Adding Destination in Segment**</br>

Once the Segment library is integrated, toggle CleverTap on in your Segment destinations, and add your CleverTap Account ID and CleverTap Account Token which you can find in the CleverTap Dashboard under Settings.

You can integrate CleverTap via a server-side or mobile destination (iOS or Andriod). If you are interested in using CleverTap’s push notifications or in-app notifications products, you should use mobile destinations.

For Adding Account Id and Account Token ->Step 1:Go to Destination in Segment -> Select CleverTap -> Configure CleverTap.

**Step 2:**</br>

Add CleverTap Account Id, Account Token and Enable the Switch from the top. For CleverTap Account Id and Account Token, Go to CleverTap Dashboard > Your Application > Settings.

**Integration in Android Application**</br>

**Step 1:** In your application Gradle file, add the following dependency

```JAVA
dependencies {
implementation 'com.clevertap.android:clevertap-segment-android:+'
implementation 'com.segment.analytics.android:analytics:4.3.1'
implementation 'com.google.firebase:firebase-messaging:17.3.4'
implementation 'com.google.android.gms:play-services-base:16.0.1'
implementation 'com.google.android.gms:play-services-ads:16.+'
implementation 'com.android.support:appcompat-v7:28.0.0'
implementation 'com.android.support:support-v4:28.0.0'
implementation 'com.android.support:design:28.0.0'//Mandatory if using App Inbox 
implementation 'com.google.android.exoplayer:exoplayer:2.8.4'//Mandatory if using App Inbox 
implementation 'com.google.android.exoplayer:exoplayer-hls:2.8.4'//Mandatory if using App Inbox 
implementation 'com.google.android.exoplayer:exoplayer-ui:2.8.4'//Mandatory if using App Inbox 
implementation 'com.github.bumptech.glide:glide:4.9.0'//Mandatory if using App Inbox
}
// at the end of the build.gradle file
apply plugin: 'com.google.gms.google-services'
```

**Step 2: To Initialize the client.**</br> 
Add the following code in your Application class file.

```JAVA
public class CleverTapSegmentApplication extends Application {

private static final String TAG = String.format("%s.%s", "CLEVERTAP", CleverTapSegmentApplication.class.getName());
private static final String WRITE_KEY = "<Your_WRITE_KEY>"; //This you will receive under source in segment. 
private static final String CLEVERTAP_KEY = "CleverTap";
public static boolean sCleverTapSegmentEnabled = false;
private CleverTapAPI clevertap;
private static Handler handler = null;

@Override 
public void onCreate() 
{
  super.onCreate();
  CleverTapAPI.setDebugLevel(CleverTapAPI.LogLevel.DEBUG);
  Analytics analytics = new Analytics.Builder(getApplicationContext(), WRITE_KEY)
    .logLevel(Analytics.LogLevel.VERBOSE) 
    .use(CleverTapIntegration.FACTORY) 
    .build();
  analytics.onIntegrationReady(CLEVERTAP_KEY, new Analytics.Callback<CleverTapAPI>() 
  { 
    @Override
    public void onReady(CleverTapAPI instance) {
        Log.i(TAG, "analytics.onIntegrationReady() called");
        CleverTapIntegrationReady(instance); 
      }
    });
  Analytics.setSingletonInstance(analytics); 
}

  private void CleverTapIntegrationReady(CleverTapAPI instance) 
  { 
    instance.enablePersonalization();
    sCleverTapSegmentEnabled = true;
    clevertap = instance;
  } 
}
```

**Step 3**: Add the following permissions in your Android Manifest file

```JAVA
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" /> <uses-permission android:name="com.google.android.c2dm.permission.RECEIVE" />
<uses-permission android:name="android.permission.WAKE_LOCK" />
```

**Step 4**: Under <application></application> tag add the following Meta Data to make CleverTap SDK to be Non-GDPR compliant.

```JAVA
<meta-data
android:name="CLEVERT AP_USE_GOOGLE_AD_ID" android:value="1"/>
```

**Step 5**: Capturing the user information 

**For Segment-CleverTap SDK version v1.1.2 and above**</br>

**Case 1**: While user login or sign-up or you wanted to push data to CleverTap (This is done to allow multiple user logins or signup on the same device)

Call Identify method of the segment which in turn will call onUserLogin of CleverTap. Example as follows

```JAVA
Traits traits = new Traits();
traits.putEmail("foo@foo.com");
traits.putName("FooName");
traits.putPhone("+14155551234"); 
Analytics.with(getApplicationContext()).identify(61026032, traits, null);
//Replace all the example values with your required dynamic ones.
```

**Note**: If you pass Email address while user Login/Sign-up, Make sure you pass this Email address whenever you wanted to update any user property to cleverTap.

**For Segment-CleverTap SDK version v1.1.1 and below**</br>

**Case 1**: While user login or signup (This is done to allow multiple user logins or signup on the same device)

First, call OnUserLogin and then Identify(segment method). Example as follows

```JAVA
HashMap<String, Object> profileUpdate = new HashMap<String, Object>(); profileUpdate.put("Name", " FooName"); // String
profileUpdate.put("Identity", 61026032); // String or number
profileUpdate.put("Email", " foo@foo.com"); // Email address of the user profileUpdate.put("Phone", "+14155551234"); // Phone (with the country code, starting with +)
CleverTapAPI.getInstance(getApplicationContext()).onUserLogin(profileUpdate);

Traits traits = new Traits();
traits.putEmail("foo@foo.com");
traits.putName("FooName");
traits.putPhone("+14155551234"); 
Analytics.with(getApplicationContext()).identify(61026032, traits, null);
//Replace all the example values with your required dynamic ones.
```

**Case 2**: Updating User Properties once the user is login or signup

Call directly Segment method to push user data. Example as follows
```JAVA
Traits traits = new Traits();
traits.putName("FooName");
traits.putGender("male");
traits.putPhone("+14155551234"); 
traits.putEmployees(123456778); 
Analytics.with(getApplicationContext()).identify(null, traits, null);
```

**CleverTap User Profile Consideration**</br>

In a User Profile, you can set a maximum number of 256 custom attribute keys

User Profile attribute keys must be of type String and attribute values can be scalar values, i.e. String,

Boolean, Integer, Float or a Date object or multi-values (returned as a JSONArray or NSArray/Array).

Attribute key names are limited to 120 characters in length.

Scalar attribute values are limited to 512 characters in length.

**Multi-value Property Constraints**</br>

Multi-values property values must be unique for that key.

Multi-value property values must be Strings and are limited to 100 items. Excess items will be removed on a

FIFO (first-in, first-out) basis.

Multi-value property values are limited to 512 characters in length.

When setting a multi-value property, any existing value will be overwritten.

When adding item(s) to a multi-value property, if the property does not exist it will be created. If the key

currently contains a scalar value, the key will be promoted to a multi-value property, with the current value

cast to a string and the new value(s) added.

When removing item(s) from a multi-value property, if the key currently contains a scalar value, prior to

performing the remove operation, the key will be promoted to a multi-value property with the current value cast to a string. If the multi-value property is empty after the remove operation, the key will be removed.

**Step 5**: Capturing Events

```JAVA
Analytics.with(getApplicationContext()).track("testEvent",
new Properties().putValue("value", "testValue") .putValue("testDate", new Date(System.currentTimeMillis()))
);
```

Here **testEvent** is Event Name and Value and **testDate** is the Event Properties

**CleverTap Event Consideration**</br>

The maximum number of User Event types per app is **512**. While the number might seem to limit, if used alongside properties can help you record a lot more User Event data than it seems. The volume of events submitted per account across those event types is practically unlimited.

For each User Event recorded, the maximum number of **Event Properties** is limited to **256**.

‘Charged’ Event supports up to **256 Items values**

Event property keys must be of type String and property values must be scalar values, i.e. String, Boolean, Integer, Float or a Date object.

**Prohibited characters**: &, $, “, \, %, >, <, !

User Event keys are limited to **120 characters in length**.

User Event property values are limited to **512 characters in length**.

**Step 5:5.A.** Enabling Push Notifications

**Case 1**: Only CleverTap is present i.e. When you are going to send Push only from CleverTap

Add the following code in your Android Manifest file

```JAVA
<service android:name="com.clevertap.android.sdk.FcmTokenListenerService">
 <intent-filter>
  <action android:name="com.google.firebase.INSTANCE_ID_EVENT"/> 
</intent-filter>
</service>

<service android:name="com.clevertap.android.sdk.FcmMessageListenerService"> 
  <intent-filter>
    <action android:name="com.google.firebase.MESSAGING_EVENT"/> 
  </intent-filter>
</service>

<meta-data
android:name="CLEVERTAP_NOTIFICATION_ICON" 
android:value="ic_stat_red_star"/>

<service android:name="com.clevertap.android.sdk.CTNotificationIntentService" >
  <intent-filter>
    <action android:name="com.clevertap.PUSH_EVENT"/> 
  </intent-filter>
</service>
```

**Case 2**: When you are going to send Push from different tools like Firebase

**Step A**: Add the following code in your Android Manifest file
```JAVA
<meta-data
  android:name="CLEVERTAP_NOTIFICATION_ICON" 
  android:value="ic_stat_red_star"/>
<!—Above meta data is optional -->

<service android:name="com.clevertap.android.sdk.CTNotificationIntentService" >
  <intent-filter>
    <action android:name="com.clevertap.PUSH_EVENT"/> 
  </intent-filter>
</service>
```

**Step B**: If your Firebase Messenger version is 18.0 and above then add the following code in your onNewToken method else under onTokenRefresh method

```JAVA
String fcmRegId = FirebaseInstanceId.getInstance().getToken(); 
clevertapDefaultInstance.pushFcmRegistrationId(fcmRegId,true);
```

**Step C**: In your MyFcmMessageListenerService class file, place the following code

```JAVA
public class MyFcmMessageListenerService extends FirebaseMessagingService {
@Override
public void onMessageReceived(RemoteMessage message){
try {
  if (message.getData().size() > 0) {
        Bundle extras = new Bundle();
        for (Map.Entry<String, String> entry : message.getData().entrySet()) 
        {
            extras.putString(entry.getKey(), entry.getValue()); 
            }
        NotificationInfo info = CleverTapAPI.getNotificationInfo(extras);
        if (info.fromCleverTap) 
        { 
          CleverTapAPI.createNotification(getApplicationContext(), extras);
        } else {
        // not from CleverTap handle yourself or pass to another provider
        } 
    }
  } catch (Throwable t) {
  Log.d("MYFCMLIST", "Error parsing FCM message", t);
  }
} 
}
````

**5.A.** Creating Notification Channel for Android O and above.

```JAVA
CleverTapAPI cleverTapAPI = CleverTapAPI.getDefaultInstance(getApplicationContext()); 
cleverTapAPI.createNotificationChannel(getApplicationContext(),"YourChannelId","Your Channel Name","Your Channel Description",NotificationManager.IMPORT ANCE_MAX,true);
// Creating a Notification Channel With Sound Support - Example 
cleverTapAPI.createNotificationChannel(getApplicationContext(),"got","Game of Thrones", "Game Of Thrones", NotificationManager.IMPORTANCE_MAX, true, "gameofthrones.mp3");
```
In the above example -> Notification sound file “gameofthrones.mp3” needs to be present in the android resource folder

**Step 6**: Enabling In-App Notification and Push Amplification services.

By default, all the required methods are present in the Segment-CleverTap SDK. So no extra declaration is required.

**Related articles**</br>
https://developer.clevertap.com/docs/android-quickstart-guide   
https://segment.com/docs/destinations/clevertap/   


