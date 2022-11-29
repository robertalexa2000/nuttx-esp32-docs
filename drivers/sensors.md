# Sensor device drivers

The software architecture of the sensor drivers is organized into two parts (two source files), each with its own data structure:

* `struct sensor_lowerhalf_s`, defined in [include/nuttx/sensors/sensor.h:870](https://github.com/apache/nuttx/blob/master/include/nuttx/sensors/sensor.h#L870)
* `struct sensor_upperhalf_s`, defined in [drivers/sensors/sensor.c:92](https://github.com/apache/nuttx/blob/master/drivers/sensors/sensor.c#L92)

## Struct sensor_lowerhalf_s

It's a generic data structure meant to be used by each sensor driver according to its needs. Thus, it contains:

* the sensor type, for example SENSOR_TYPE_LIGHT. It's used for defining the name of the character device.
    * sensor_light, sensor_temp, sensor_humi, etc.
* depending on the sensor's operating mode (polling or interrupt-driven), the API can define a circular buffer of size N to hold the data retrieved from the sensor. The lowerhalf part stores only the number of entries in this buffer.
* whether the sensor is calibrated or not - affecting only the sensor's character device name
    * sensor_light0 vs sensor_light_uncal0
* a pointer to `struct sensor_ops_s` which in turn stores pointers to functions. These callbacks are initialized by the driver and used for interacting with the sensor's internals.

## Struct sensor_upperhalf_s

This data structure is the bridge between the user and the sensor. It contains:

* a pointer to `struct sensor_lowerhalf_s` corresponding to the sensor it designates
* a circular buffer for storing the data
* a lock for sequential access
* a list of all the users who opened the sensor's char device - the first user enables the sensor, whereas the last user to close the file descriptor disables the sensor

## Sensor.c interface

The existing support in NuttX makes writing sensor drivers very easy. As explained above, the interaction between the user and the sensor is done entirely through the API provided by `sensor.c`:

* the callbacks of open, close, read, write, ioctl, poll are predefined. Thus, the ioctl commands are all generic and can be adapted to each sensor's needs without writing new ones.
* the buffer's management is transparent to the sensor driver - the driver either solely "pushes" the information through a callback (polling) or notifies the upperhalf that an event has occurred (interrupt-driven)

## What should a driver do?

1. First and foremost, define a set of callbacks for interacting with the sensor's internals.
    * A minimal set could consist of **activate** and **calibrate** (for devices that use polling).
    * If the sensor is working in the interrupt-driven mode, then the driver must also define the **fetch** callback.
1. Afterwards, a `struct sensor_lowerhalf_s` data structure must be initialized.
1. In order to register the sensor as char device, the driver must call `sensor_register` (defined in `sensor.c`). This will register the character driver under `/dev/uorb/`.
1. Finally, a kthread is created and started.
    * If the sensor is working by polling, the thread will wait for the user to open the char device and in this way enable the sensor. Then, whenever new data is available the thread will push it to the upperhalf by using the **push_event** callback.
    * If the interrupts are enabled, the data will be transmitted through the **notify_event** callback.

PLease refer to [LTR308](https://github.com/robertalexa2000/incubator-nuttx/blob/master/drivers/sensors/ltr308.c) driver implementation for devices that work by polling and to [L3GD20](https://github.com/robertalexa2000/incubator-nuttx/blob/master/drivers/sensors/l3gd20.c) for implementing the interrupt-driven mechanism.
