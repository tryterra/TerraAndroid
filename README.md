# TerraAndroid

This library allows developers to connect to all integrations provided by [Terra](https://tryterra.co) in one easy to install library!

Docs are updated to newest version: 1.3.x

For docs on older versions please see [here](https://github.com/tryterra/TerraAndroid/blob/79b2c265cfc6292d52cb96dc36e1a3438200471c/README.md)

## The boring stuff
This library was built on Android Studio Artic Fox with Kotlin 1.6 (jvmTarget 1.8). This package targets a minimum SDK version of 26 (Android OREO). 

## The useful yet boring stuff

This library can be broken into two parts: 
- Samsung Health, GoogleFit and FreeStyleLibre in-app integrations
- REST API integrations

### For GOOGLE FIT or SAMSUNG HEALTH

You will need to install [Health Connect](https://play.google.com/store/apps/details?id=com.google.android.apps.healthdata).

Then for Samsung Health:
Settings -> Health Connect -> Enable permissions for Samsung Health to Write/Read from Health Connect

For Google Fit:
Settings -> Health Connect ->  Enable permissions for Google Fit to Write/Read from Health Connect

You will also need to include the following in your AndroidManifest:
```
<intent-filter>
    <action android:name="androidx.health.ACTION_SHOW_PERMISSIONS_RATIONALE" />
</intent-filter>
```

To the Activity you wish to show when the user checks on your privacy policy page for Health Connect. 

```
 <meta-data
     android:name="health_permissions"
     android:resource="@array/health_permissions" />
```
To the activity you wish to request permissions and use Health Connect from. 
