## Building the IoT client ##

1. Now we will start to code the actual IoT Client.
As most TV Chefs we don't start from scratch, we have a template to start from.

Move to the bin directory of the project
```
cd iotcs/posix/bin
```
 And copy the template to the file we will use as per below.
```
cp iotclient_template.c iotclient.c
*An important note here. If you skipped ahead an already ran the iotclient.c then you must continue work with the exact same file. You can open and edit it, replacing the contents in it etc. But do not replace it via a copy operation.*
```
2. Open the file **iotclient.c** either on the Raspberry or if you choose to pull down the repo to your laptop.

3. Now we will start to add the required API's to the client. First lets look at what we got from the template.

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <time.h>
#include "pi_2_dht_read.h"

/* include common public types */
#include "iotcs.h"
/* include iot cs device model APIs */
#include "iotcs_virtual_device.h"
/* include methods for device client*/
#include "iotcs_device.h"

/* Device model handle */
static iotcs_device_model_handle device_model_handle = NULL;
/* Device handle */
static iotcs_virtual_device_handle device_handle = NULL;

/* print error message and terminate the program execution */
static void error(const char* message) {
    fprintf(stderr,"iotcs: Error occurred: %s\n", message);
    exit(EXIT_FAILURE);
}

/*
** Define const Variables
*/
// Set sensor type DHT11=11, DHT22=22
const int sensor_type = 22;
// The sensor is on GPIO pin=4
const int gpio_pin = 4;

/************************************************************************************************
** Main
************************************************************************************************/
int main(int argc, char** argv) {
    /* This is the URN of your device model. */
    const char* device_urns[] = {
        "urn:com:oracle:demo:esensor",
        NULL
    };

    if (argc < 3) {
        error("Too few parameters.\n"
                "\nUsage:"
                "\n\tiotclient.out path password"
                "\n\tpath is a path to trusted assets store."
                "\n\tpassword is a password for trusted assets store.");
    }
    const char* ts_path = argv[1];
    const char* ts_password = argv[2];

	// Define Variables
    iotcs_result rv;
	int i = 0;
	int result = -1;
	float humidity = 0;
	float temperature = 0;


  fprintf(stderr,"iotcs: iotclient starting!\n");
  fprintf(stderr,"iotcs: Device Urn: %s\n", device_urns[0]);
  fprintf(stderr,"iotcs: Loading configuration from: %s\n", ts_path);


/*
*
*
*
*
*
* Your code here!
*
*
*
*
*
*/

    return EXIT_SUCCESS;
}
```

You can try and compile the code already to see that everything is in place. We have prepared a shell script you can use. To build run the script as below
```
sh build_iotclient.sh
```

4. If you haven't already, you now want to **copy** your downloaded **provisioning file** (xyz.conf) to the Raspberry, to the bin directory. Either add it to your repository, or use FileZilla/sftp/similar.

**Update** the **_run_iotclient.sh_** to use the name of your provisioning file and password.

5. Now run the client.
```
sh run_iotclient.sh
```

This should return something similar to this:
```
iotcs: iotclient starting!
iotcs: Device Urn: urn:com:oracle:demo:esensor
iotcs: Loading configuration from: /home/pi/iotcs/posix/bin/NameOfProvisioningFile.conf
```

4. Some explanation of the template. The first important const variable declaration is the type of sensor. The DHT11 and DHT22 need different drivers so it is important to tell which type we are using.
```
// Set sensor type DHT11=11, DHT22=22
const int sensor_type = 22;
```
5. Next we are telling ** which pin** we have connected the sensor on the GPIO connector.
```
// The sensor is on GPIO pin=4
const int gpio_pin = 4;
```
6. **Time to code**. Next we have a value that we need to set according to our team name. Remember your **URN** from when you first created the device model in the IoT Cloud Service? It is the string "urn:com:oracle:demo:esensor" that needs to be changed into "urn:com:discotechoracle:demo:**TeamName"**. This is the way that the IoT server will recognize which device models that our device supports.
```
int main(int argc, char** argv) {
    /* This is the URN of your device model. */
    const char* device_urns[] = {
        "urn:com:oracle:demo:esensor",
        NULL
    };
```

7. First we need to Initialize the IoT library. **Add this code** to iotclient.c where indicated by the comment "Add your code here!"

```
/*
 * Initialize the library before any other calls.
 * Initiate all subsystems like ssl, TAM, request dispatcher,
 * async message dispatcher, etc which needed for correct library work.
 */

if (iotcs_init(ts_path, ts_password) != IOTCS_RESULT_OK) {
    error("Initialization failed");
}
```

8. Next we need to activate the device if it is not already activated. **Add this code** to iotclient.c after the previously entered section.
```
/*
 * Activate the device, if it's not already activated.
 * Always check if the device is activated before calling activate.
 * The device model URN is passed into the activate call to tell
 * the server the device model(s) that are supported by this
 * device
 */

if (!iotcs_is_activated()) {
    if (iotcs_activate(device_urns) != IOTCS_RESULT_OK) {
        error("Sending activation request failed");
    }
}
```
9. Get the handles needed to be able to call the IoT library API's. **Add this code** to iotclient.c after the previously entered section.
```
/* get device model handle */
if (iotcs_get_device_model_handle(device_urns[0], &device_model_handle) != IOTCS_RESULT_OK) {
    fprintf(stderr,"iotcs_get_device_model_handle method failed\n");
    return IOTCS_RESULT_FAIL;
}

/* get device handle */
if (iotcs_get_virtual_device_handle(iotcs_get_endpoint_id(), device_model_handle, &device_handle) != IOTCS_RESULT_OK) {
    fprintf(stderr,"iotcs_get_device_handle method failed\n");
    return IOTCS_RESULT_FAIL;
}
```
10. Now lets read some values from the DHT sensor! This API **pi_2_dht_read()** comes from the DHT client library. We have also added some logging to be able to see what is going on at the client side. **Add this code** to iotclient.c after the previously entered section.
```
// Read values from the sensor.
fprintf(stderr,"iotcs: Reading from the DHT%u sensor!\n", sensor_type);
result = pi_2_dht_read(sensor_type, gpio_pin, &humidity, &temperature);
if (result != DHT_SUCCESS) {
  fprintf(stderr,"iotcs: Warning, Bad data from the DHT%u sensor\n", sensor_type);
}
fprintf(stderr,"iotcs: temperature = %2.2f, humidity= %2.2f\n", temperature, humidity);
```
11. Here is an important API you need to add. It tells IoT that you want to "open an update session" on the client. This means that after this you can add multiple attributes to send and IoT will wait for you to tell it when it is ok to do so. In a couple of rows we will do just that. Add this code to iotclient.c after the previously entered section.
```
// Start setting attribute for IOT
iotcs_virtual_device_start_update(device_handle);
```
12. Now lets set some attributes that we want to send to the IoT server. This is of course the temperature and humidity values we just read from the sensor. Please note that we are now changing attributes on the **"digital twin"**. The IoT infrastructure will keep the server and the client in sync. There is no need to actually "send" the data. Add this code to iotclient.c after the previously entered section.
```
// Set attribute
rv = iotcs_virtual_device_set_float(device_handle, "temperature", temperature);
if (rv != IOTCS_RESULT_OK) {
  fprintf(stderr,"iotcs_virtual_device_set_float 1 failed\n");
  return IOTCS_RESULT_FAIL;
}

// Set attribute
rv = iotcs_virtual_device_set_float(device_handle, "humidity", humidity);
if (rv != IOTCS_RESULT_OK) {
  fprintf(stderr,"iotcs_virtual_device_set_float 2 failed\n");
  return IOTCS_RESULT_FAIL;
}
```
13. Now that the attributes are all set we can close the "update session" and let IoT sync the data to the server.  Add this code to iotclient.c after the previously entered section.
```
// We are done. IOT can sync the virtual device
iotcs_virtual_device_finish_update(device_handle);
```
14. And finally release the device handles and call the Finalize API. This will close all communication etc.  Add this code to iotclient.c after the previously entered section.
```
/* free device handle */
iotcs_free_virtual_device_handle(device_handle);
/* free device model handle */
iotcs_free_device_model_handle(device_model_handle);

/*
 * Calling finalization of the library ensures communications channels are closed,
 * previously allocated temporary resources are released.
 */
iotcs_finalize();
```

Now we will test that the client actually works!

### [Test the client](testiotclient.md) ###
