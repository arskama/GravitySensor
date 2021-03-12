# GravitySensor Interface

## Introduction
The **GravitySensor API** provides the capability
to get a 3-axis reading of the gravity force to users.

See also:
* [The current specification](https://w3c.github.io/accelerometer/#gravitysensor-interface)
* [Motion Sensors explainer](https://w3c.github.io/motion-sensors/)
* [web.dev article](https://web.dev/generic-sensor/#gravity-sensor)

## Goals
It is possible for users to manually derive readings close
to those of a gravity sensor by manually inspecting Accelerometer
and LinearAccelerometer readings, but this can be cumbersome and depend
on the accuracy of the values provided by those sensors.
Platforms such as Android can provide gravity readings as part of the operating system,
which should be cheaper in terms of computation, provide more accurate values depending
on the user's hardware, and be easier to use in terms of API ergonomics.
The GravitySensor returns the effect of acceleration along the device's X, Y, and Z axis due to gravity.

## API
The following example shows how to use GravitySensor to get readings
in the screen coordinate system. The snippet will print message to the console
when the device screen is perpendicular to the ground and bottom 
of the rendered web page is pointing downwards.

```js
let sensor = new GravitySensor({frequency: 5, referenceFrame: "screen"});

sensor.addEventListener('reading', e => {
  if (sensor.y >= 9.8) {
    console.log("Web page is perpendicular to the ground.");
  }
});

sensor.start();
```
The following example implements a GravitySensor polyfill. It shows the complexity of extracting the gravity value from the Accelerometer and LinearAccelerationSensor interfaces, 
in case both sensors are hardware-based sensors. In case there is no hardware-based LinearAccelerationSensor, 
Chrome will implement some fusion logic based on the Accelerometer value and provide a less accurate value.
In the example, a simplification shortcut has been made to listen only to accelerometer events and not to both sensors' events.
```js
class GravitySensor extends EventTarget {
  #accelerometer = new Accelerometer();
  #linearAccelerationSensor = new LinearAcceleration();
  x = 0;
  y = 0;
  z = 0;

  handleEvent(ev) {
    this.timestamp = ev.timestamp;
    this.x = this.#accelerometer.x - this.#linearAccelerationSensor.x;
    this.y = this.#accelerometer.y - this.#linearAccelerationSensor.y;
    this.z = this.#accelerometer.z - this.#linearAccelerationSensor.z;
    const event = new Event("reading");
    this.dispatchEvent(event);
    this.onreading?.(event);
  }

  start() {
    this.#accelerometer.addEventListener("reading", this);
    this.#accelerometer.start();
  }

  stop() {
    this.#accelerometer.removeEventListener("reading", this);
    this.#accelerometer.stop();
  }
}

g = new GravitySensor();
g.onreading = () => console.log(g.x, g.y, g.z);
g.start();
```
It is also possible to isolate directly the gravity from the accelerometer value, using a low-pass filter.
```js
// In this example, alpha is calculated as t / (t + dT),
// where t is the low-pass filter's time-constant and
// dT is the event delivery rate.
const alpha = 0.8;
gravity.x = alpha * gravity.x + (1 - alpha) * accelerometer.x;
gravity.y = alpha * gravity.y + (1 - alpha) * accelerometer.y;
gravity.z = alpha * gravity.z + (1 - alpha) * accelerometer.z;
```

Other examples can be found in [the current specification](https://w3c.github.io/accelerometer/#examples)

## Use cases

Many causal mobile games which don't use touch input use a gravity sensor value to:
* Steering a rolling ball through a maze (either tilting the maze, or changing the ball's direction)
* Turn a vehicle by tilting the device left and right through traffic.
* Creating a "spirit-level" balance game where you have to balance objects by adjusting the device to change the gravity reading.
* Observing changes in the gravity sensor reading can indicate a swing motion of the device for a golf game

## Security considerations
* Security considerations are already described in [the current specifications](https://w3c.github.io/accelerometer/#security-and-privacy)
* The Chrome implementation performs rounding of all sensor readings to avoid fingerprinting issues. As with Accelerometer and LinearAccelerometer, the GravitySensor readings are rounded to 0.1m/s^2

## Stakeholder Feedback / Opposition
* Web developers: Positive, [Unity](https://unity.com/) has filled a [bug](https://crbug.com/1163993) for Chromium
* Chromium: [Positive](https://chromestatus.com/features/5384099747332096)
* Edge: No Signal
* Mozilla: Negative
* Safari: [Negative](https://webkit.org/tracking-prevention/)
