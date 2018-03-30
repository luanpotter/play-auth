# How to use Google Play Games auth?
## Intro

Doing this is a hassle; i've tryed for many days to make it work. This is a consolidation of my results.

Following this: https://developers.google.com/games/services/console/enabling

-- todo play console vs cloud console

## Setup

Create blank project with android-studio

generate keystore:

```bash
    keytool -genkey -v -keystore my-release-key.keystore -alias alias_name -keyalg RSA -keysize 2048 -validity 10000
```

Put password
Name and UNKNOWN (enter all the way) -> yes
enter for the same password


then get sha1
keytool -list -v -keystore test.keystore -alias test -storepass 123test -keypass 123test 2> /dev/null | grep "SHA1:" | rex '\s*SHA1: (.*)' '$1'

## Step 1. Sign in to the Google Play Console

Go [here](https://play.google.com/apps/publish/); if you are not logged in with Google, do so. If you've never accessed Google Play Console, you might need to accept some terms.

You will see something like this:

![1](images/1_play_console.png)

Then, click on `Game Services` on the left sidebar (you can open it clicking on the hamburger icon), and then on the `ADD NEW GAME` blue button near the top right corner.

![2](images/2_play_console_games.png)

-- todo i already use google cloud console

Select `I don't use any Google APIs in my game yet`; add the name of your game and the category of your choosing. These will have no effects on the rest of the process.

![3](images/3_play_console_new_game.png)

Linked apps -> Android -> Save and continue
Authorize app now -> put sha1 -> get client id


then:

ids files + 
```xml
<meta-data android:name="com.google.android.gms.games.APP_ID"
    android:value="524310031182" />
<meta-data android:name="com.google.android.gms.version"
    android:value="@integer/google_play_services_version"/>
```


gradle
deps internal
    implementation "com.google.android.gms:play-services-games:${gms_library_version}"
    implementation "com.google.android.gms:play-services-auth:${gms_library_version}"
root external

ext {
    gms_library_version = '11.6.2'
}
