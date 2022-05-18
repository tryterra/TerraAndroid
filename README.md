# TerraAndroid

This library allows developers to connect to Samsung Health, GoogleFit, and FreestyleLibre1 through Terra on Androids! It also contains necessary functions and classes to connect to the Terra REST API if so needed!

## The boring stuff
This library was built on Android Studio Artic Fox with Kotlin 1.6 (jvmTarget 1.8). This package targets a minimum SDK version of 28 (Android PIE). 

## The useful yet boring stuff

As mentioned, this library has functionality to connect to Samsung Health, Google Fit and FreeStyleLibre1

### For Samsung Health

Please make sure the users have downloaded [Health Platform](https://play.google.com/store/apps/details?id=com.samsung.android.service.health&hl=en_GB&gl=US) on their devices and linked their Samsung Health Account to Health Platform. This can be done by going on Samsung Health -> Profile -> Settings -> Connected Services -> Health Platform and giving Health Platform access to their data. 

**N.B Samsung Health Feature is only available for SELECT Samsung Devices (Android OREO)+**

### For Google Fit

For Google Fit, please register your project package name from your `AndroidManifest.xml` file with [Google API](https://console.cloud.google.com). You will need to create a new project and a new set of Oauth credentials within this page. When creating the project, you will also need to consider scopes needed for the project. Enable Fitness API and People API, and select the following scopes:

**People API**
- openid
- /auth/user.birthday.read
- /auth/user.emails.read
- /auth/user.gender.read
- /auth/userinfo.email
- /auth/userinfo.profile

**Fitness API**
- /auth/fitness.activity.read
- /auth/fitness.blood_glucose.read
- /auth/fitness.blood_pressure.read
- /auth/fitness.body.read
- /auth/fitness/heart_rate.read
- /auth/fitness.body_temperature.read
- /auth/fitness.location.read
- /auth/fitness.nutrition.read
- /auth/fitness.oxygen_saturation.read
- /auth/fitness.reproductive_health.read
- /auth/fitness.sleep.read

Finish creating the OAuth Credentials as required and you should be able to start developing with Google Fit!

### For FreeStyleLibre

Nothing. Really ;)

## Installation

Download the `.aar` file from this repository and include it within the `app/libs` folder in your project structure. You may now add it as a dependency in your gradle configuration files. In your project level gradle file (`build.gradle(:Project)`), edit the `repositories` to include:

```gradle
flatDir{
  dirs 'libs'
}
```

For example:
```gradle
repositories {
    google()
    mavenCentral()
    flatDir {
        dirs 'libs'
    }
}
```

Then you must include the library and the coroutines library in your app level gradle file(`build.gradle(:app)`) by adding the following lines under `dependencies`:

```gradle
implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-core-jvm:1.5.2'
implementation files('libs/TerraAndroid-alpha.aar')

```

For example:
```gradle
dependencies {
    implementation 'androidx.core:core-ktx:1.7.0'
    implementation 'androidx.appcompat:appcompat:1.4.0'
    implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-core-jvm:1.5.2'
    implementation files('libs/TerraAndroid-alpha.aar')
}
```

You may now import classes from the library as: `import co.tryterra.terra.(Every class in this library imaginable (except private ones))`

## Using the library!

To connect to Samsung Health, Google Fit, or FreeStyleLibre1, you will need to instantiate a Terra Class as follows:

```kotlin
val terra: Terra = terra = Terra(
                devId = <YOUR DEV ID>,
                XAPIKey = <YOUR X API KEY>,
                context = this, //Your App Activity
                bodyTimer = 60 * 60 * 1000,
                dailyTimer = 60 * 60 * 1000,
                sleepTimer = 60 * 60 * 1000,
                nutritionTimer = 60 * 60 * 1000,
                activityTimer = 60 * 60 * 1000,
                referenceId = "testingRef",
            )
```

- The `bodyTimer`, `dailyTimer`, `sleepTimer`, `nutritionTimer`, and `activityTimer` arguments are the intervals at which the data for body, daily, sleep, nutrition, and activity are going to be scheduled and sent to your webhook. Defaults to 8 hours for every timer except activity for which is 20 minutes.
- The `referenceId` argument is a unique identifer for you to use in order to map the Terra User ID to your own. Defaults to null


You will now be able to initialise any providers you wish using:

```kotlin
terra!!.initConnection(connection = Connections.SAMSUNG, context = this, samsungHealthPermissions = setOf(SamsungHealthpermissions.ACTIVITY), googleFitPermissions = setOf(GoogleFitPermissions.DAILY, startIntent: String?)
```

- The `connection` argument takes a `Connection` from `co.tryterra.terra.Connections`. This signifies the connection you wish to make through Terra. There are currently 3 connections you could make: FREESTYLE_LIBRE, SAMSUNG, and GOOGLE_FIT
- The `samsungHealthPermissions` argument takes a `SetOf<SamsungHealthPermissions>` from `co.tryterra.terra.samsung.SamsungHealthPermissions`. It signifies the data types you wish to request permissions for in Samsung Health. This defaults to all permissions included.
- The `googleFitPermissions` argument takes a `SetOf<GoogleFitPermissions>` from `co.tryterra.terra.googlefit.GoogleFitPermissions`. It signifies the data types you wish to request permissions for in Google Fit. This defaults to all permissions included.
- The `startIntent` argument takes an Optional (defaults to `null`) String. It signifies the Activity for which you want to start after a FreeStyleLibre Sensor scan is complete. For example if your package name is (in your `AndroidManifest.xml`) is `co.tryterra.terrademo`, and the activity you wish to start after the scan is complete is called `MainActivity`, then you would insert: `co.tryterra.terrademo.MainActivity`. 

**N.B Running this function automatically brings up permission and login screens! 

You may then check the Terra `user_id`'s with the following function:

```kotlin
terra!!.getUserId(Terra.Connections)
```

## Getting Data 

You can get data either from the scheduled intervals you set, or by manual queries.

### Body Data

```kotlin
terra.getBody(type: Terra.Connections, startDate: Date, endDate: Date)
```

### Sleep Data

```kotlin
terra.getSleep(type: Terra.Connections, startDate: Date, endDate: Date)
```

### Daily Data

```kotlin
terra.getDaily(type: Terra.Connections, startDate: Date, endDate: Date)
```

### Activity Data

```kotlin
terra.getActivity(type: Terra.Connections, startDate: Date, endDate: Date)
```

### Nutrition Data

```kotlin
terra.getNutrition(type: Terra.Connections, startDate: Date, endDate: Date)
```

### Athlete Data

```kotlin
terra.getAthlete(type: Terra.Connections)
```

The `startDate` and `endDate` parameters here can take both `Date` and `Long` arguments! (`Long` for UNIX Timestamp since 1st Jan 1970)

These functions will automatically send the data to your webhooks. However if needed you may also catch the data from the callback function provided:

```kotlin
terra.getBody(type: Terra.Connections, startDate: Date, endDate: Date){success, payload ->
            // success -> If the function executed successfully
            // payload -> Data following our [models](https://docs.tryterra.co/data-models-mark-ii)
        }
```

### FreeStyleLibre Specifics

For FreeStyleLibre, you simply have to scan your sensor (only FreeStyleLibre1 is supported at the moment) and the data will be sent directly to your webhook! Please note you will need to wait for 2 vibrations!

### Deauthentication

You can easily disconnect a user by:

```kotlin
terra.disconnect(Terra.Connections)
```

## Connection to REST API

This package also allows you to call the "authenticateUser" and "deauthenticateUser" endpoint along with the data request endpoints provided by [Terra API](https://docs.tryterra.co). 

### Authenticate User and Deauthenticate User

The `TerraAuthClient` class deals with these two endpoints. First you have to initiate one:

```kotlin
val terraAuthClient: TerraAuthClient = TerraAuthClient("YOUR X API KEY", "YOUR DEV ID")
```

and then call the following function:
```kotlin
terraAuthClient.authenticateUser(RESOURCE){it ->
    //Do something with the response
        Log.i(TAG, it.toString())
    }
```
Call back returns an `AuthenticateUser` datamodel with the following fields:

```kotlin
data class AuthenticateUser(
    var status: String,
    var auth_url: String,
    var user_id: String,
)
```

Similarly for deauthentication:

```kotlin
terraAuthClient.deauthenticateUser(USER_ID)
```

### Data Endpoints

To use the Terra API Data endpoints, you may do so with the `TerraClient` class:

```kotlin
val terraClient = TerraClient("USER ID", "YOUR X API KEY", "YOUR DEV ID")
```

Using this class, you can then request for data. The following example requests Body Data from our API with startDate and endDate parameters set as UNIX Timestamps in seconds. You may set "toWebhook" to false if you wish for the callback function to return the data payload.

```kotlin
    terra.getBody(
        startDate = Date.from(Instant.ofEpochMilli(startTime)).toInstant().epochSecond,
        endDate = Date.from(Instant.ofEpochMilli(endTime)).toInstant().epochSecond,
        toWebhook = false
    ){bodyData ->
        // Do something with body data
        Log.i(TAG, bodyData.toString())
    }
```

The other data functions would be : `getDaily`, `getActivity`, `getSleep`, `getNutrition`. These functions uses the same argument as the `getBody` function and all of them returns a callback with their own datamodels. The `toWebhook` argument defaults to `true` if not provided.

The `startDate` and `endDate` parameters here can take both `Date` and `Long` arguments! (`Long` for UNIX Timestamp since 1st Jan 1970)

The data models correspond to having fields (exactly) the same as we the ones given by our [Data Models](https://docs.tryterra.co/data-models-mark-ii)

Finally theres also `getAthlete` for which only accepts `toWebhook` argument.


