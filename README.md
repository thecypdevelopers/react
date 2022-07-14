The Cyprinus - React Native &middot;
============================================================================


-------------------------------------------------------------------------------

El módulo *más* sofisticado de **seguimiento de ubicación y geoperimetraje** en segundo plano con inteligencia de detección de movimiento consciente de la batería para **iOS** y **Android**.

**detección de movimiento** (usando acelerómetro, giroscopio y magnetómetro) para detectar cuando el dispositivo se *mueve* y *estacionario*.

- Cuando se detecta que el dispositivo **se está moviendo**, el complemento *automáticamente* comenzará a registrar una ubicación de acuerdo con el `distanceFilter` (metros) configurado.

- Cuando se detecta que el dispositivo está **estacionario**, el complemento desactivará automáticamente los servicios de ubicación para conservar energía.


----------------------------------------------------------------------------

# Contenido
- ### [Instalación de Plugin](#large_blue_diamond-installing-the-plugin)
- ### [Setup](#large_blue_diamond-setup-guides)
- ### [Ingresar la licencia](#large_blue_diamond-configure-your-license)
- ### [Uso del plugin](#large_blue_diamond-using-the-plugin)
- ### [Ejemplo](#large_blue_diamond-example)
- ### [Debug](../../wiki/Debugging)
- ### [Applicación Demo](#large_blue_diamond-demo-application)
- ### [Testeo del server](#large_blue_diamond-simple-testing-server)

## :large_blue_diamond: Instalación del plugin

-------------------------------------------------------------

```bash
$ react-native unlink react-native-background-geolocation
```

-------------------------------------------------------------

### With `yarn`

```bash
yarn add react-native-background-geolocation
```

### With `npm`
```
$ npm install react-native-background-geolocation --save
```

## :large_blue_diamond: Setup

### `react-native >= 0.60`

### iOS
- [Auto-linking Setup](help/INSTALL-IOS-AUTO.md)

### Android
- [Auto-linking Setup](help/INSTALL-ANDROID-AUTO.md)


## :large_blue_diamond: Activar la licencia

1. The Cyprinus enviará el codigo de licencia por un medio seguro:

2. Agregar linencia en  `android/app/src/main/AndroidManifest.xml`:

```diff
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.transistorsoft.backgroundgeolocation.react">

  <application
    android:name=".MainApplication"
    android:allowBackup="true"
    android:label="@string/app_name"
    android:icon="@mipmap/ic_launcher"
    android:theme="@style/AppTheme">

    <!-- react-native-background-geolocation licence -->
+     <meta-data android:name="com.transistorsoft.locationmanager.license" android:value="YOUR_LICENCE_KEY_HERE" />
    .
    .
    .
  </application>
</manifest>
```

## :large_blue_diamond: Uso del plugin ##

```javascript
import BackgroundGeolocation from "react-native-background-geolocation";
```

### [Typescript](https://facebook.github.io/react-native/blog/2018/05/07/using-typescript-with-react-native) API:

For those using [Typescript](https://facebook.github.io/react-native/blog/2018/05/07/using-typescript-with-react-native) (__recommended__), you can also `import` the interfaces:
```javascript
import BackgroundGeolocation, {
  State,
  Config,
  Location,
  LocationError,
  Geofence,
  GeofenceEvent,
  GeofencesChangeEvent,
  HeartbeatEvent,
  HttpEvent,
  MotionActivityEvent,
  MotionChangeEvent,
  ProviderChangeEvent,
  ConnectivityChangeEvent
} from "react-native-background-geolocation";

```

## :large_blue_diamond: Ejemplo

There are three main steps to using `BackgroundGeolocation`
1. Wire up event-listeners.
2. `.ready(config)` the plugin.
3. `.start()` the plugin.

:warning: Do not execute *any* API method which will require accessing location-services until the **`.ready(config)`** method resolves (eg: `#getCurrentPosition`, `#watchPosition`, `#start`).

```javascript
// NO!  .ready() has not resolved.
BackgroundGeolocation.getCurrentPosition(options);
BackgroundGeolocation.start();

BackgroundGeolocation.ready(config).then((state) => {
  // YES -- .ready() has now resolved.
  BackgroundGeolocation.getCurrentPosition(options);
  BackgroundGeolocation.start();  
});

// NO!  .ready() has not resolved.
BackgroundGeolocation.getCurrentPosition(options);
BackgroundGeolocation.start();
```


### Ejemplo 1. &mdash; React *Functional Component*

<details>
  <summary>Ver Código</summary>

```javascript

import React from 'react';
import {
  Switch,
  Text,
  View,
} from 'react-native';

import BackgroundGeolocation, {
  Location,
  Subscription
} from "react-native-background-geolocation";

const HelloWorld = () => {
  const [enabled, setEnabled] = React.useState(false);
  const [location, setLocation] = React.useState('');

  React.useEffect(() => {
    /// 1.  Subscribe to events.
    const onLocation:Subscription = BackgroundGeolocation.onLocation((location) => {
      console.log('[onLocation]', location);
      setLocation(JSON.stringify(location, null, 2));
    })

    const onMotionChange:Subscription = BackgroundGeolocation.onMotionChange((event) => {
      console.log('[onMotionChange]', event);
    });

    const onActivityChange:Subscription = BackgroundGeolocation.onActivityChange((event) => {
      console.log('[onMotionChange]', event);
    })

    const onProviderChange:Subscription = BackgroundGeolocation.onProviderChange((event) => {
      console.log('[onProviderChange]', event);
    })

    /// 2. ready the plugin.
    BackgroundGeolocation.ready({
      // Geolocation Config
      desiredAccuracy: BackgroundGeolocation.DESIRED_ACCURACY_HIGH,
      distanceFilter: 10,
      // Activity Recognition
      stopTimeout: 5,
      // Application config
      debug: true, // <-- enable this hear sounds for background-geolocation life-cycle.
      logLevel: BackgroundGeolocation.LOG_LEVEL_VERBOSE,
      stopOnTerminate: false,   // <-- Allow the background-service to continue tracking when user closes the app.
      startOnBoot: true,        // <-- Auto start tracking when device is powered-up.
      // HTTP / SQLite config
      url: 'http://yourserver.com/locations',
      batchSync: false,       // <-- [Default: false] Set true to sync locations to server in a single HTTP request.
      autoSync: true,         // <-- [Default: true] Set true to sync each location to server as it arrives.
      headers: {              // <-- Optional HTTP headers
        "X-FOO": "bar"
      },
      params: {               // <-- Optional HTTP params
        "auth_token": "maybe_your_server_authenticates_via_token_YES?"
      }
    }).then((state) => {
      setEnabled(state.enabled)
      console.log("- BackgroundGeolocation is configured and ready: ", state.enabled);
    });

    return () => {
      // Remove BackgroundGeolocation event-subscribers when the View is removed or refreshed
      // during development live-reload.  Without this, event-listeners will accumulate with
      // each refresh during live-reload.
      onLocation.remove();
      onMotionChange.remove();
      onActivityChange.remove();
      onProviderChange.remove();
    }
  }, []);

  /// 3. start / stop BackgroundGeolocation
  React.useEffect(() => {
    if (enabled) {
      BackgroundGeolocation.start();
    } else {
      BackgroundGeolocation.stop();
      setLocation('');
    }
  }, [enabled]);

  return (
    <View style={{alignItems:'center'}}>
      <Text>Click to enable BackgroundGeolocation</Text>
      <Switch value={enabled} onValueChange={setEnabled} />
      <Text style={{fontFamily:'monospace', fontSize:12}}>{location}</Text>
    </View>
  )
}

export default HelloWorld;
```

</details>

### Ejemplo 2. &mdash; React *Class Component*

<details>
  <summary>Ver Código</summary>

```javascript
import React from 'react';
import {
  Switch,
  Text,
  View,
} from 'react-native';

import BackgroundGeolocation, {
  Location,
  Subscription
} from "react-native-background-geolocation";

export default class HelloWorld extends React.Component {
  subscriptions:Subscription[] = [];
  state:any = {};
  constructor(props:any) {
    super(props);
    this.state = {
      enabled: false,
      location: ''
    }
  }

  componentDidMount() {
    /// 1.  Subscribe to BackgroundGeolocation events.
    this.subscriptions.push(BackgroundGeolocation.onLocation((location) => {
      console.log('[onLocation]', location);
      this.setState({location: JSON.stringify(location, null, 2)})
    }, (error) => {
      console.log('[onLocation] ERROR:', error);
    }))

    this.subscriptions.push(BackgroundGeolocation.onMotionChange((event) => {
      console.log('[onMotionChange]', event);
    }))

    this.subscriptions.push(BackgroundGeolocation.onActivityChange((event) => {
      console.log('[onActivityChange]', event);
    }))

    this.subscriptions.push(BackgroundGeolocation.onProviderChange((event) => {
      console.log('[onProviderChange]', event);
    }))

    /// 2. ready the plugin.
    BackgroundGeolocation.ready({
      // Geolocation Config
      desiredAccuracy: BackgroundGeolocation.DESIRED_ACCURACY_HIGH,
      distanceFilter: 10,
      // Activity Recognition
      stopTimeout: 5,
      // Application config
      debug: true, // <-- enable this hear sounds for background-geolocation life-cycle.
      logLevel: BackgroundGeolocation.LOG_LEVEL_VERBOSE,
      stopOnTerminate: false,   // <-- Allow the background-service to continue tracking when user closes the app.
      startOnBoot: true,        // <-- Auto start tracking when device is powered-up.
      // HTTP / SQLite config
      url: 'http://yourserver.com/locations',
      batchSync: false,       // <-- [Default: false] Set true to sync locations to server in a single HTTP request.
      autoSync: true,         // <-- [Default: true] Set true to sync each location to server as it arrives.
      headers: {              // <-- Optional HTTP headers
        "X-FOO": "bar"
      },
      params: {               // <-- Optional HTTP params
        "auth_token": "maybe_your_server_authenticates_via_token_YES?"
      }
    }).then((state) => {
      this.setState({enabled: state.enabled});
      console.log("- BackgroundGeolocation is configured and ready: ", state.enabled);
    })
  }

  /// When view is destroyed (or refreshed during development live-reload),
  /// remove BackgroundGeolocation event subscriptions.
  componentWillUnmount() {
    this.subscriptions.forEach((subscription) => subscription.remove());
  }

  onToggleEnabled(value:boolean) {
    console.log('[onToggleEnabled]', value);
    this.setState({enabled: value})
    if (value) {
      BackgroundGeolocation.start();
    } else {
      this.setState({location: ''});
      BackgroundGeolocation.stop();
    }
  }

  render() {
    return (
      <View style={{alignItems:'center'}}>
        <Text>Click to enable BackgroundGeolocation</Text>
        <Switch value={this.state.enabled} onValueChange={this.onToggleEnabled.bind(this)} />
        <Text style={{fontFamily:'monospace', fontSize:12}}>{this.state.location}</Text>
      </View>
    )
  }
}
```
</details>

### Promise API

The `BackgroundGeolocation` Javascript API supports [Promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) for *nearly* every method (the exceptions are **`#watchPosition`** and adding event-listeners via **`#onXXX`** method (eg: `onLocation`).

```javascript
BackgroundGeolocation.ready({
  desiredAccuracy: BackgroundGeolocation.DESIRED_ACCURACY_HIGH, 
  distanceFilter: 50
}).then(state => {
  console.log('- BackgroundGeolocation is ready: ', state);
}).catch(error => {
  console.warn('- BackgroundGeolocation error: ', error);
});

// Or use await in an async function
try {
  const state = await BackgroundGeolocation.ready({
    desiredAccuracy: BackgroundGeolocation.DESIRED_ACCURACY_HIGH, 
    distanceFilter: 50
  })
  console.log('- BackgroundGeolocation is ready: ', state);
} catch (error) {
  console.warn('- BackgroundGeolocation error: ', error);
}
```

## :large_blue_diamond: Demo Application

   
Una aplicación de demostración con todas las funciones está disponible. Esta aplicación de demostración incluye una pantalla de configuración que le permite experimentar rápidamente con todas las diferentes configuraciones disponibles para cada plataforma. Solicitar a The Cyprinus.    
    

![Home](https://dl.dropboxusercontent.com/s/wa43w1n3xhkjn0i/home-framed-350.png?dl=1)
![Settings](https://dl.dropboxusercontent.com/s/8oad228siog49kt/settings-framed-350.png?dl=1)


## :large_blue_diamond: Conexión al servidor de prueba

Solicitar accesos a The Cyprinus

