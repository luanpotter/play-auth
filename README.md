# How to use Google Play Games auth?
## Intro

Doing this is a hassle; i've tryed for many days to make it work. This is a consolidation of my results.

Following this: https://developers.google.com/games/services/console/enabling

WIP: IT HASNT WORKED YET!

## Concepts

### Play Console vs Cloud Console

Google has two consoles: the play console is related to publishing games on the Play Store and using the Google Play Games. Cloud Console is the regular console for anything Google Cloud related (GAE, GCE, IAM, etc, etc). Don't get those mixed up!

## Setup

Firstly, create a blank project with android-studio (you can use their wizards dialog for that).

Select a package name and save it somewhere (you will need it later). Make sure these three places match the same package name:

 * The folder structure (app/src/main/java/your/package/name);
 * The actual java files (`package your.package.name`);
 * Your AndroidManifest (app/src/main/AndroidManifest.xml), in the `manifest` tag, the `package` property; something like:

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android" package="your.package.name">
```

As per Java convention, you should always select a package name that represent a domain that your own (but reversed); e.g., com.yorudomain.subdomain.anything.

Then, you must generate a new keystore; just run:

```bash
    keytool -genkey -v -keystore my-release-key.keystore -alias alias_name -keyalg RSA -keysize 2048 -validity 10000
```

Put a password of your choosing. This is a not a personal password of yours, it's just to encrypt this file, that you might share with other people on the project. So just generate a random one and keep it in a secure place for safekeeping.

Then you need to fill a lot of information. Normally I fill only my name (first name) and leave everything else UNKNOWN. Notice that in the end you must explicitly write [yes] to accept everything.

For the second password, just press enter to use the same as before (you could have two, but that'd be too much, I guess).

Finally, you need to get the SHA1 key associated with the key you generated; for that, just run:

```bash
    keytool -list -v -keystore test.keystore -alias test -storepass 123test -keypass 123test 2> /dev/null | grep "SHA1:" | rex '\s*SHA1: (.*)' '$1'
```

After generating the key, you must setup your build.gradle file to use the key to sign your app. You will need to put the SHA1 you generated in the Play Console for configuration, and only an app signed with this key will work. Even in debug mode. Therefore, add the following for your internal build.gradle file (app/build.gradle):

```groovy
    signingConfigs {
        debug {
            storeFile file("/home/luan/projects/play-auth/secret/test.keystore")
            storePassword '123test'
            keyAlias 'test'
            keyPassword '123test'
        }
    }
    buildTypes {
        debug {
            signingConfig signingConfigs.debug
        }
    }
```

Be sure to put the path to your key (better if it's a relative path, but never commit the key itself to a public repo), the passwords you selected, and the key alias (must be the same).

This will add the key just for debugging; for release, add another build type named release (you probably already have one, so just add the signingConfig property. You can use the same key, but it's ideal to have two (so that you can share the debug with everyone in your team and keep the release one very well secured).

## Step 1. Sign in to the Google Play Console

Go [here](https://play.google.com/apps/publish/); if you are not logged in with Google, do so. If you've never accessed Google Play Console, you might need to accept some terms.

You will see something like this:

![1](images/1_play_console.png)

Then, click on `Game Services` on the left sidebar (you can open it clicking on the hamburger icon), and then on the `ADD NEW GAME` blue button near the top right corner.

![2](images/2_play_console_games.png)

Select `I don't use any Google APIs in my game yet`; add the name of your game and the category of your choosing. These will have no effects on the rest of the process.

![3](images/3_play_console_new_game.png)

Then go to:

> Linked apps -> Android -> Save and continue

Then

> Authorize app now -> put your SHA1 and package name

Here you can get your clientId and appId; save those somewhere safe.

> appId: 13 digits number

> clientId: appId + some random string + ".apps.googleusercontent.com"

## Step 2. Link your android project to the account you created

Create a ids file (app/src/main/res/values/ids.xml), with the following content:

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <item name="app_id" type="string" value="your app id" />
</resources>
```

Then edit your manifest file (app/src/main/AndroidManifest.xml) and add the following lines (WHERE?):

```xml
<meta-data android:name="com.google.android.gms.games.APP_ID" android:value="@string/app_id" />
<meta-data android:name="com.google.android.gms.version" android:value="@integer/google_play_services_version"/>
```

Add the following to your gradle file (build.gradle) (WHICH ONE?):

```grovy
deps internal
    implementation "com.google.android.gms:play-services-games:${gms_library_version}"
    implementation "com.google.android.gms:play-services-auth:${gms_library_version}"
root external

ext {
    gms_library_version = '11.6.2'
}
```
