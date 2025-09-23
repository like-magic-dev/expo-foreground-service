Works only on Android. Battle tested with thousands of devices and users.

This needs the notification permissions to be granted before using it. Will silently fail if there are no permissions, this module does NOT ask for permissions, it should be handled elsewhere in your code, for example: 

```ts
  if (isAndroid) {
    if (Number(Platform.Version) >= 33) {
      await PermissionsAndroid.request(
        PermissionsAndroid.PERMISSIONS.POST_NOTIFICATIONS
      ).catch((e) => console.log(e));
    }
  }
```

#### hook usage:

```ts
  const { startForegroundService, stopForegroundService } = useForegroundNotification(
    {
        alert: false,
        onServiceErrorCallBack: () => {
            console.warn('Foreground service error');
        }
    },
    {
        ServiceType: 'dataSync',
        icon: 'ic_launcher',
        importance: 'low',
        message: 'App will stay alive during capture',
        setOnlyAlertOnce: true,
        title: 'Camera Active',
        visibility: 'secret'
    }
);
```

Make the appropriate changes in `AndroidManifest.xml` like the color and the channel name for the notification. 


I am too lazy to create an npm module for this so just copy paste in your `modules` folder. 


More info on expo-modules can be found here: https://docs.expo.dev/modules/overview/ and https://docs.expo.dev/modules/native-module-tutorial/


No further documentation, sorry. It's straight forward, though. Just check the [types](https://github.com/efstathiosntonas/expo-foreground-service/blob/main/modules/foreground-service/src/ForegroundService.types.ts) and the [class](https://github.com/efstathiosntonas/expo-foreground-service/blob/main/modules/foreground-service/src/ForegroundService.ts) 😂
