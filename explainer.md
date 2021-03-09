# GravitySensor Interface

## Introduction
The **GravitySensor API** provides, the capability
to get a 3-axis reading of the gravity force to users.

See also:
* [The current specification](https://w3c.github.io/accelerometer/)
* [web.dev article](https://web.dev/generic-sensor/#gravity-sensor)

## Goals
It is already possible for users to manually derive readings close
to those of a gravity sensor by manually inspecting Accelerometer
and LinearAccelerometer readings, but this can be cumbersome and depend
on the accuracy of the values provided by those sensors.
Platforms such as Android can provide gravity readings as part of the operating system,
which should be cheaper in terms of computation, provide more accurate values depending
on the user's hardware, and be easier to use in terms of API ergonomics.
The GravitySensor returns the effect of acceleration along the device's X, Y, and Z axis due to gravity.

## API
The following example shows how to use gravity sensor that provides readings
in the screen coordinate system. The snippet will print message to the console
when the dom screen is perpendicular to the ground and bottom of the rendered web page
is pointing downwards.

```js
let sensor = new GravitySensor({frequency: 5, referenceFrame: "screen"});

sensor.onreading = () => {
  if (sensor.y >= 9.8) {
    console.log("Web page is perpendicular to the ground.");
  }
}

sensor.start();
```
Other examples can be found in [the current specification](https://w3c.github.io/accelerometer/#examples)

## Use cases

**TO BE FILLED**

## Security considerations
* Security considerations are already described in [the current specifications](https://w3c.github.io/accelerometer/#security-and-privacy)
* To protect user privacy, in chrome, rounding sensor values to multiples of a reasonably small value is performed.
This makes the readings slightly less accurate, but protects users from fingerprinting attacks which
might allow a site to generate a unique id based on the sensor calibration data.

## Stakeholder Feedback / Opposition
* Web developers: Positive, [Unity](https://unity.com/) has filled a [bug](https://crbug.com/1163993) for Chromium
* Chromium: Positive
* Edge: No Signal
* Mozilla: Negative
* Safari: Negative
