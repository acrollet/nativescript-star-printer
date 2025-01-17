# NativeScript Star Printer

[![NPM version][npm-image]][npm-url]
[![Downloads][downloads-image]][npm-url]
[![Twitter Follow][twitter-image]][twitter-url]

[build-status]:https://travis-ci.org/EddyVerbruggen/nativescript-star-printer.svg?branch=master
[build-url]:https://travis-ci.org/EddyVerbruggen/nativescript-star-printer
[npm-image]:http://img.shields.io/npm/v/nativescript-star-printer.svg
[npm-url]:https://npmjs.org/package/nativescript-star-printer
[downloads-image]:http://img.shields.io/npm/dm/nativescript-star-printer.svg
[twitter-image]:https://img.shields.io/twitter/follow/eddyverbruggen.svg?style=social&label=Follow%20me
[twitter-url]:https://twitter.com/eddyverbruggen

<img src="https://github.com/EddyVerbruggen/nativescript-star-printer/raw/master/media/demo-app.gif" width="328px" height="577px" />

_That's [the demo app](https://github.com/EddyVerbruggen/nativescript-star-printer/tree/master/demo) in action, printing on a Star Micronics TSP650II_

## Installation
```bash
tns plugin add nativescript-star-printer
```

## API

### requiring / importing the plugin
All examples below assume you're using TypeScript, but here's how to require the plugin with plain old JS as well:

#### JavaScript
```js
var StarPrinterPlugin = require("nativescript-star-printer");
var starPrinter = new StarPrinterPlugin.StarPrinter();
```

#### TypeScript
```typescript
import { StarPrinter, SPPrinter, SPCommands } from "nativescript-star-printer";

export Class MyPrintingClass {
  private starPrinter: StarPrinter;
  
  constructor() {
    this.starPrinter = new StarPrinter();
  }
}
```

### `searchPrinters`
If you're searching for a Bluetooth printer, enable Bluetooth in the device settings
and pair/connect the printer. Then do:

```typescript
this.starPrinter.searchPrinters().then(
    (printers: Array<SPPrinter>) => {
      console.log(`Found ${printers.length} printers`);
    }, (err: string) => {
      console.log(`Search printers error: ${err}`);
    });
```

The most useful property on the `SPPrinter` class is the `portName` which you will need
in other API methods.

The only other property is `modelName`.

### `connect`
Once you know the printer port name, you can connect to it.

> Note that there's no need to connect if you want to print as the print function does this automatically.

```typescript
this.starPrinter.connect({
  portName: thePortName
}).then((result: SPPrinterStatusResult) => console.log("Connected: " + result.connected));
```

### `getPrinterStatus`
After connecting to a printer, you can use this method to poll for the 'online' and 'paper' statuses.

```typescript
this.starPrinter.getPrinterStatus({
  portName: this.lastConnectedPrinterPort
}).then(result => {
  const online: boolean = result.online;
  const onlineStatus: PrinterOnlineStatus = result.onlineStatus;
  const paperStatus: PrinterPaperStatus = result.paperStatus;
});
```

### `print`
Once you've got the port of the printer you want to print to, just do:

```typescript
this.starPrinter.print({
  portName: this.selectedPrinterPort,
  commands: commands
});
```

So what are those `commands`? Let's recreate the fake receipt below to answer that (see the TypeScript definition for all options):

<img src="https://github.com/EddyVerbruggen/nativescript-star-printer/raw/master/media/demo-app-receipt-with-barcode.jpg" width="500px" />

```typescript
const image = ImageSource.fromFile("~/res/mww-logo.png");

// Note that a standard 3 inch roll is 48 characters wide - we use that knowledge for our "columns"
let commands = new SPCommands()
    .image(
        image,
        true, // diffuse
        true // align center (set to 'false' to align left)
     )
    .alignCenter()
    .text("My Awesome Boutique").newLine()
    .text("In a shop near you").newLine()
    .setFont("smaller")
    .text("Planet Earth").newLine()
    .setFont("default")
    .newLine()
    .text("Date: 11/11/2017                   Time: 3:15 PM")
    .horizontalLine()
    .newLine()
    .textBold("SKU           Description                  Total").newLine()
    .text("300678566     Plain White Tee             €10.99").newLine()
    .text("300692003     Black Dénim                 €29.99").newLine()
    .text("300651148     Blue Denim                  €29.99").newLine()
    .newLine()
    .newLine()
    .barcode({
      type: "Code128",
      value: "12345678",
      width: "large",
      height: 60,
      appendEncodedValue: false
    })
    .newLine()
    .cutPaper();

this.starPrinter.print({
  portName: this.selectedPrinterPort,
  commands: commands
});
```

### `openCashDrawer`
In case a cash drawer is connected via the UTP (network) connector of the Star printer,
you can open the drawer from your code!

```typescript
this.starPrinter.openCashDrawer({
  portName: this.selectedPrinterPort
});
```

## iOS runtime permission reason
iOS 10+ requires a permission popup when connecting (the first) time to a Bluetooth peripheral explaining *why* it needs to connect.

You can provide your own reason by adding something like this to `app/App_Resources/ios/Info.plist`:

```xml
  <key>NSBluetoothPeripheralUsageDescription</key>
  <string>My reason justifying fooling around with your Bluetooth</string>
```

_To not crash your app in case you forgot to provide the reason this plugin adds an empty reason to the `.plist` during build. This value gets overridden by anything you specified yourself. You're welcome._

## Known limitations
On iOS you want to run this on a real device.


## Future work
Possibly add more `print` formatting options.
