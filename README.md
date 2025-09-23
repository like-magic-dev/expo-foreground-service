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

real life usage scenario: 

```ts
import { useCallback } from 'react';

import { Asset } from '#/typings';
import {
  getCameraPermissionsAsync,
  requestCameraPermissionsAsync
} from 'expo-image-picker';

import { checkCameraPermission } from '@functions/image';

import { useForegroundNotification } from '@hooks/useForegroundNotification';

import { cameraPermissionsError } from '@utils/commonAlerts';
import { crop, openCamera } from '@utils/picker';
import { sleep } from '@utils/utils';

type CameraOptions = {
  errorContext?: string;
  onError?: (e: any) => void;
  skipCropOnAndroid?: boolean;
};

/**
 * A hook for handling camera operations with consistent error handling
 * @returns A function to open camera and process the image
 */
export const useCamera = () => {
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

  const handleCameraCapture = useCallback(
    async <T>(
      onSuccess: (image: Asset) => T | Promise<T>,
      options: CameraOptions = {}
    ): Promise<T | undefined> => {
      const { errorContext = 'CameraCapture', onError } = options;

      try {
        await startForegroundService();
        /* Add small delay to let foreground service stabilize */
        await sleep(100);
        await checkCameraPermission();
        const { status } = await getCameraPermissionsAsync();
        if (status !== 'granted') {
          const perm = await requestCameraPermissionsAsync();
          if (perm.status !== 'granted') {
            throw new Error('Required permission not granted');
          }
        }
        const img = await openCamera();

        /* Add delay after camera capture, similar to gallery */
        await sleep(500);

        const croppedImage = await crop(img.uri);

        if (croppedImage) {
          return await onSuccess(croppedImage);
        }

        return undefined;
      } catch (e: any) {
        console.log(e);
        if (onError) {
          onError(e);
        } else {
          cameraPermissionsError(e, errorContext);
        }
        return undefined;
      } finally {
        await stopForegroundService();
      }
    },
    [startForegroundService, stopForegroundService]
  );

  return { handleCameraCapture };
};

```


Make the appropriate changes in `AndroidManifest.xml` like the color and the channel name for the notification. 


I am too lazy to create an npm module for this so just copy paste in your `modules` folder. 


More info on expo-modules can be found here: https://docs.expo.dev/modules/overview/ and https://docs.expo.dev/modules/native-module-tutorial/


No further documentation, sorry. It's straight forward, though. Just check the [types](https://github.com/efstathiosntonas/expo-foreground-service/blob/main/modules/foreground-service/src/ForegroundService.types.ts) and the [class](https://github.com/efstathiosntonas/expo-foreground-service/blob/main/modules/foreground-service/src/ForegroundService.ts) 😂
