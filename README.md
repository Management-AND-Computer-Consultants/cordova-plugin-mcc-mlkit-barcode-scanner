## cordova-plugin-mcc-mlkit-barcode-scanner

Google ML Kit barcode scanning for Cordova (Android and iOS). Ships a simple JS API and TypeScript types, supports multiple barcode formats, optional beep/vibration feedback, and a configurable detector window.

- **Platforms**: Android, iOS
- **Cordova**: `cordova >= 7.1.0`, `cordova-android >= 8.0.0`, `cordova-ios >= 4.5.0`


### Install

```bash
cordova plugin add cordova-plugin-mcc-mlkit-barcode-scanner
```


### iOS permission
Add a camera permission description (required by iOS) to your app’s `config.xml`:

```xml
<edit-config file="*-Info.plist" mode="merge" target="NSCameraUsageDescription">
  <string>Camera access is required to scan barcodes.</string>
</edit-config>
```


### Global object
The plugin is exposed on the Cordova namespace after `deviceready`:

```
cordova.plugins.mlkit.barcodeScanner
```


### API

- `scan(options, success, failure): void`
  - `options` (partial, all optional):
    - `barcodeFormats`: object of booleans to enable formats:
      - `Aztec`, `CodaBar`, `Code39`, `Code93`, `Code128`, `DataMatrix`, `EAN8`, `EAN13`, `ITF`, `PDF417`, `QRCode`, `UPCA`, `UPCE`
    - `beepOnSuccess`: boolean (default `false`)
    - `vibrateOnSuccess`: boolean (default `false`)
    - `detectorSize`: number between 0 and 1, relative detector height (default `0.6`)
    - `rotateCamera`: boolean (default `false`)
  - `success(result)`:
    - `result.text`: string (raw barcode text)
    - `result.format`: string (one of the enabled formats above)
    - `result.type`: string (ML Kit content type, e.g. `TEXT`, `URL`, `WIFI`, …)
  - `failure(error)`:
    - `error.cancelled`: boolean
    - `error.message`: string (`"The scan was cancelled."`, `"Scanner already open."`, or error message)

Default options enable all formats, with beep/vibrate off.


### Basic usage (vanilla JS)

```javascript
document.addEventListener('deviceready', function () {
  var scanner = cordova.plugins.mlkit.barcodeScanner;

  var options = {
    barcodeFormats: { QRCode: true, Code128: true }, // others use defaults
    beepOnSuccess: true,
    vibrateOnSuccess: true,
    detectorSize: 0.6, // 60% of view height
    rotateCamera: false
  };

  scanner.scan(
    options,
    function onSuccess(result) {
      console.log('Text:', result.text);
      console.log('Format:', result.format);
      console.log('Type:', result.type);
    },
    function onFailure(err) {
      if (err && err.cancelled) {
        console.log('User cancelled');
      } else {
        console.error('Scan failed:', err && err.message);
      }
    }
  );
});
```


### AngularJS / Ionic 1

Factory wrapper:

```javascript
angular.module('app').factory('BarcodeScanner', function ($q, $window, $ionicPlatform) {
  function getScanner() {
    return $window.cordova
      && $window.cordova.plugins
      && $window.cordova.plugins.mlkit
      && $window.cordova.plugins.mlkit.barcodeScanner;
  }

  function scan(options) {
    return $q(function (resolve, reject) {
      $ionicPlatform.ready(function () {
        var scanner = getScanner();
        if (!scanner) {
          return reject({ cancelled: false, message: 'Plugin not available' });
        }
        scanner.scan(options || {}, resolve, reject);
      });
    });
  }

  return { scan: scan };
});
```

Controller usage:

```javascript
angular.module('app').controller('ScanCtrl', function ($scope, BarcodeScanner) {
  $scope.scan = function () {
    BarcodeScanner.scan({
      barcodeFormats: { QRCode: true, Code128: true },
      beepOnSuccess: true
    }).then(function (res) {
      $scope.result = res.text;
    }).catch(function (err) {
      if (!err.cancelled) {
        $scope.error = err.message || 'Scan failed';
      }
    });
  };
});
```


### TypeScript (optional)

```ts
import type { IOptions, IResult, IError } from 'cordova-plugin-mcc-mlkit-barcode-scanner';

document.addEventListener('deviceready', () => {
  const options: IOptions = {
    barcodeFormats: { QRCode: true },
  };

  cordova.plugins.mlkit.barcodeScanner.scan(
    options,
    (r: IResult) => console.log(r.text, r.format, r.type),
    (e: IError) => console.error(e.message),
  );
});
```


### Notes
- Call the API only after `deviceready` (or `$ionicPlatform.ready`).
- Only one scanner can be open at a time; attempting to open another returns `"Scanner already open."`.
- Android permissions are added automatically; iOS requires `NSCameraUsageDescription` as shown above.


### License
MIT


