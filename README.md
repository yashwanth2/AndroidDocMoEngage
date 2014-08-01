#MoEngageAndroid

MoEngage Documentation for Android.

##Adding Jar file to the Android project

Add the given MoEngage-Android-SDK.jar file to the project, by copying the jar file to the libs folder. If there is no libs folder please create a libs folder. Make sure that you have the latest android support jar file (android-support-v4.jar) accessable to the project.

##Push Registration
create a Google API project :

1. Open the [Google Developers Console](https://cloud.google.com/console).
2. If you haven't created an API project yet, click Create Project.
3. Supply a project name and click Create.
4. Once the project has been created, a page appears that displays your project ID and project number. For example, Project Number: 670330094152.
5. Copy down your project number. You will use it later on as the GCM sender ID.

Enabling the GCM Service

1. In the sidebar on the left, select APIs & auth.
2. In the displayed list of APIs, turn the Google Cloud Messaging for Android toggle to ON.

Obtaining an API Key

1. In the sidebar on the left, select APIs & auth > Credentials.
2. Under Public API access, click Create new key.
3. In the Create a new key dialog, click Server key.
4. In the resulting configuration dialog, leave the box empty and Click Create.
5. In the refreshed page, copy the API key and send it to us.

##Change the manifest file
To enable push notifications, add the following lines to your manifest file by replacing PACKAGE_NAME with your package name, you have to replace the package name 3 times.

  	<permission android:name="PACKAGE_NAME.permission.C2D_MESSAGE" android:protectionLevel="signature" />
	<uses-permission android:name="PACKAGE_NAME.permission.C2D_MESSAGE" /> 
	<uses-permission android:name="com.google.android.c2dm.permission.RECEIVE" />
	<uses-permission android:name="android.permission.GET_ACCOUNTS" />
	<uses-permission android:name="android.permission.WAKE_LOCK"/>
	
	<receiver android:name="com.moe.pushlibrary.PushGcmBroadcastReceiver"
    		android:permission="com.google.android.c2dm.permission.SEND" >
    		<intent-filter>
      			<action android:name="com.google.android.c2dm.intent.RECEIVE" />
      			<action android:name="com.google.android.c2dm.intent.REGISTRATION" />
      			<category android:name="PACKAGE_NAME" />
    		</intent-filter>
  	</receiver>
  	<service android:name="com.moe.pushlibrary.PushGCMIntentService" />
  	<receiver android:name="com.moe.pushlibrary.PushGcmRegister" />
  	<receiver android:name="com.moe.pushlibrary.SendReport" />
  	
To add Install Attribution (User Acquisition Source) tracking, add the following lines

  	<receiver android:name="com.moe.pushlibrary.InstallReceiver">
		<intent-filter>
			<action android:name="com.android.vending.INSTALL_REFERRER"/>
    		</intent-filter>
  	</receiver>
  
Gcm ids are refreshed after every update, to handle that please put the following code, note that the PACKAGE_NAME has to be replaced with your app pakage name.

	<receiver android:name="com.moe.pushlibrary.PushUpdateReceiver">
	<intent-filter>
        	<action android:name="android.intent.action.PACKAGE_REPLACED" />
        	<data android:path="PACKAGE_NAME"
              	android:scheme="package" />
    	</intent-filter>
	</receiver>

##Initialize your app with MoEngage
Put the following code in the first activity onCreate() method

    MoEHelper mHelper = new MoEHelper(this);
    mHelper.initialize("GCM Sender ID", "MoEngage APP ID");
    
- GCM Sender ID   - the ID of the project created as part of Push Registration
- MoEngage APP ID - This is an application specific id, which MoEngage team must have shared with you.

Put the following code after the above initialization code to register for push

    mHelper.Register(drawableResourceId);
    drawableResourceId - for eg. R.drawable.icon
    
##Tracking Event
Every event has 2 attributes, action name and key, value pairs which represent additional information about the action. Add all the additional information which you think would be useful for segmentation while creating campaigns.
For eg. the following code tracks a purchase event of a product.

    JSONObject newJson = new JSONObject();
		try {
		  newJson.put("product", "Moto E");
		  newJson.put("purchase value", 7000);
		  newJson.put("currency", "Rs.");
    } catch (JSONException e) {
				// json exception
		}
	MoEHelper.getInstance(mCurrentContext).trackEvent("purchase", newJson);
		
	mCurrentContext - context instance
	
To pass location as one of the parameters for the event use the following code:

    MoEHelperUtils.setLocation(newJson, "attribute name", lat, lng);
    1st argument - json object which contains all the parameters for the event
    2nd argument - attribute name that you want to assign to the location
    3rd,4th - latitude and longitude of a location.
    for eg.
    JSONObject newJson = new JSONObject();
		try {
		  newJson.put("city", "New York");
		  MoEHelperUtils.setLocation(newJson, "city search", 40.77, 73.98);
    } catch (JSONException e) {
				// json exception
		}
	MoEHelper.getInstance(mCurrentContext).trackEvent("search", newJson);
		
##Setting user attributes
Use the following lines to set User attributes like Name, Email, Mobile, Gender, etc.

For eg. to set unique id for the user

    MoEHelper.getInstance(mCurrentContext).setUserAttribute(MoEHelperConstants.USER_ATTRIBUTE_UNIQUE_ID, uniqueId);
    uniqueId - unique id for the user specific to your system, so that there is a unique identifier mapping between your platform and MoEngage.
    
You can use MoEHelperConstants class to set the default user attributes like mobile number, gender, user name, brithday. Birthday has to be in the format - "mm/dd/yyyy". The constants for these default attributes in MoEHelperConstants are mentioned below:

    USER_ATTRIBUTE_UNIQUE_ID
    USER_ATTRIBUTE_USER_EMAIL
    USER_ATTRIBUTE_USER_MOBILE
    USER_ATTRIBUTE_USER_NAME   # incase you have full name 
    USER_ATTRIBUTE_USER_GENDER
    USER_ATTRIBUTE_USER_FIRST_NAME # incase you have first and last name separately
    USER_ATTRIBUTE_USER_LAST_NAME
    USER_ATTRIBUTE_USER_BDAY
    GENDER_MALE = "male";
    GENDER_FEMALE = "female";

to set user email

    MoEHelper.getInstance(mCurrentContext).setUserAttribute(MoEHelperConstants.USER_ATTRIBUTE_USER_EMAIL, email);
    email - email of the user

To set user location, use the following line

    MoEHelper.getInstance(mCurrentContext).setUserLocation(lat, lng);
    lat - latitude of the location
    lng - longitude of the location

##Setting custom user attributes
The above examples demonstrate how to set predefined attributes and their values. To set custom attributes use the following syntax.

    MoEHelper.getInstance(mCurrentContext).setUserAttribute(key, value);
    key - the name you want to give to the attribute
    value - the value you would like to assign to it

##Tracking user activity
Put the following code in every activity of the app

	MoEHelper.getInstance(this).onStart(this); - in onStart()
	MoEHelper.getInstance(this).onStop(this);  - in onStop()

as shown in the codes below

	protected void onStart() {
		super.onStart();
		MoEHelper.getInstance(this).onStart(this);
	}
	protected void onStop() {
		super.onStop();
		MoEHelper.getInstance(this).onStop(this);
	}

##Landing Page requirements
Put the following activity as part of manifest


	<activity 
		android:name="com.moe.pushlibrary.activities.MoEActivity"
		android:parentActivityName="yourparentactivityname" 
	>
		<meta-data
			android:name="android.support.PARENT_ACTIVITY"
			android:value="yourparentactivityname" 
		/>
	</activity>
       	
parent activity name is needed if you want to redirect the user to a particular screen after users clicks on the up button.If you don't want to add the parent activity just include the following lines.

	<activity 
		android:name="com.moe.pushlibrary.activities.MoEActivity" 
	>
	</activity>


##Testing event tracking after integration
To test event tracking, first you need to login to the MoEngage portal with the credentials provided for your app.

After adding event tracking in the app as shown in the guide above, you can visit "For Developers" (http://app.moengage.com/latestActivity) link through the MoEngage portal to check whether the events are being tracked, as you use.

As users use the application, events data is stored locally and sent in regular intervals of 30 seconds to avoid any performance impact. So, you might need to wait for sometime to see the events in the portal.

##Testing Push notifications after integration

You can either send the build to MoEngage team for testing the push notifications (or) follow the below process:

Please set your GCM Authorization key by visiting 'Settings' (http://app.moengage.com/account) link on the MoEngage portal to test the push notifications.

You can create a push notification campaign through 'Create Campaign' (http://app.moengage.com/newpushcampaign).
If this is the first time you are testing MoEngage SDK, you can just set a test message, set the scheduling to run 'as soon as possible' and create the campaign.

If you have a login in your app, you can also create a campaign targeting only your device by setting the filters based on user attributes.

You should be able to receive the push notification, once the campaign is created.





