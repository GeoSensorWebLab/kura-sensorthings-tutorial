# Kura and SensorThings Tutorial

[Kura](http://www.eclipse.org/kura/) is a platform for deploying and managing sensor application packages to a device. You can create a custom Kura application to submit data to multiple data stores, or use one of the built-in cloud services.

[SensorThings](http://ogc-iot.github.io/ogc-iot-api/) is a specification for communicating sensor data from clients to a data store, and includes sensor and location metadata. In this tutorial, we will build a test application in Kura that will submit sensor data to a SensorThings API. This tutorial depends on the Kura emulator, and as such can only be run on Mac OS X or Linux. Windows users will instead have to deploy the app to an actual Kura device, which is not covered in this tutorial — the [Kura guide does include instructions on deploying packages](http://eclipse.github.io/kura/doc/hello-example.html#deploying-the-plug-in).

Some of the instructions are based on the [Kura Getting Started Guide](http://eclipse.github.io/kura/doc/kura-setup.html). All the code is also available on Github and Bitbucket — see "Source Code" section below.

This tutorial is for the April 2016 version of Kura, which is version 1.4.0.

## Set Up Java, Eclipse

We will be using Java 8 for this project, primarily due to its improved date/time libraries. You can download [Java SE 8 JDK from Oracle](http://www.oracle.com/technetwork/java/javase/downloads/index-jsp-138363.html#javasejdk), or [OpenJDK 8](http://openjdk.java.net/install/) for Linux/BSD. Download and follow the installation instructions to set up Java. Be sure to download a **JDK** and not a **JRE**, as the JDK contains both a JRE and additional tools and libraries for building Java applications.

The primary Kura development environment is Eclipse. For this tutorial I will be using Eclipse 4.5.2 Mars.2. Download "Eclipse IDE for Java EE Developers" from [the Eclipse website](http://www.eclipse.org/downloads/). If you already have a different version of Eclipse, I recommend downloading a separate copy for Kura development.

When you run Eclipse for the first time, it will ask you to create a workspace. This is a directory that will store all your projects and code. I recommend creating a new workspace just for this Kura tutorial.

![Eclipse Workspace](images/neweclipse.png)

Next we have to install mToolkit. This allows us to connect to a Kura device (or emulator) from Eclipse and monitor/control it.

Open the menu **Help | Install New Software…**. In the first field, enter `http://mtoolkit.tigris.org/updates/stable`. Three items should appear in the main view list; if they are not visible, uncheck **Group items by category**. Once the items are displayed, enable `mToolkit` and select **Next**, then follow the remaining installation instructions in Eclipse.

## Set up Kura Development Environment

Next we will import some base projects into our workspace that will allow us to work with Kura. Go to the [Kura Downloads](https://www.eclipse.org/kura/downloads.php) page and select "Developer's Workspace (with Web UI)" for version 1.4.0. There are two workspaces to download, be sure to select the one with the Web UI.

In Eclipse, select **File | Import…**. Select **General > Existing Projects into Workspace**, then select **Next**. Select an archive, and browse to the ZIP file we just downloaded. Four projects should appear, go ahead and import all of them.

The projects will appear in the workspace, but there will be some errors displayed. This is because we have not told Eclipse where we will be deploying these projects. Expand the `target-definition` project, and open the `kura-equinox_3.8.1.target` file. Select **Set as Target Platform**. All the errors in the Eclipse workspace should disappear.

![Eclipse Kura Workspace](images/changetarget.png)

## Testing the Emulator

The Kura projects we imported include an emulator for Mac OS and Linux. This runs a local instance of Kura that allows us to test a Kura app quickly in development. It gives us access to logs, making debugging easier.

First make sure you aren't running any other services on port 8080. Then expand the `org.eclipse.kura.emulator` project, and then expand the `src/main/resources` directory. Right-click the `Kura_Emulator_OSX.launch` (or `Kura_Emulator_Linux.launch`) file, and select **Run As | 1 Kura Emulator OSX/Linux**. (Note that if the "Package Explorer" is not in focus, the **Run As** menu will only show "Run Configurations…'.)

The Console tab should activate and scroll a bunch of information. At this point it either works or displays an error.

### There is a Problem

In my case I get an error:

    java.lang.UnsupportedClassVersionError: org/eclipse/kura/system/SystemAdminService : Unsupported major.minor version 52.0

Click the red stop button in the Console tab to stop the emulator. The wrong version of Java JDK is being used, we have to change that.

Right-click the `Kura_Emulator_OSX.launch` (or `Kura_Emulator_Linux.launch`) file, and select **Run As | Run Configurations…**. In the left-side pane, ensure the emulator configuration for your OS is selected. In the main view, select the **Settings** tab. Change the Execution Environment to `JavaSE-1.8 (Java SE 8 [1.8.0_73])`. (Your patch number may be slightly different — be sure to select JavaSE-1.8 though). Click the **Apply** button, then the **Run** button. You should get an output similar to the next section.

![Run Configuration](images/runconfigurations.png)

### There isn't a Problem

If the emulator launches correctly, you should see output similar to below:

    osgi> 11:57:21,745 [Start Level Event Dispatcher] INFO  Server:253  - jetty-8.1.3.v20120522
    11:57:21,796 [Start Level Event Dispatcher] INFO  AbstractConnector:333  - Started SelectChannelConnector@0.0.0.0:8080
    11:57:21,807 [Component Resolve Thread] INFO  CertificatesManager:60  - Bundle org.eclipse.kura.core.certificates.CertificatesManager has started!
    Framework is running in emulation mode
    11:57:21,953 [Component Resolve Thread] INFO  SystemServiceImpl:147  - Loaded URL kura.properties: file:/Users/jpbadger/code/workspace2/org.eclipse.kura.emulator/src/main/resources/kura.properties
    11:57:21,953 [Component Resolve Thread] INFO  SystemServiceImpl:200  - Did not locate a kura_custom.properties file in null
    11:57:21,953 [Component Resolve Thread] INFO  SystemServiceImpl:297  - Kura has net admin? false
    11:57:21,954 [Component Resolve Thread] INFO  SystemServiceImpl:300  - Kura has web interface? true
    11:57:21,954 [Component Resolve Thread] INFO  SystemServiceImpl:303  - Kura version? KURA_1.4.0
    11:57:21,954 [Component Resolve Thread] INFO  SystemServiceImpl:458  - Kura home directory is /tmp/kura
    11:57:21,954 [Component Resolve Thread] INFO  SystemServiceImpl:465  - Kura snapshots directory is /Users/jpbadger/code/workspace2/kura/snapshots
    11:57:21,954 [Component Resolve Thread] INFO  SystemServiceImpl:472  - Kura tmp directory is /tmp/kura/tmp
    11:57:21,954 [Component Resolve Thread] INFO  SystemServiceImpl:476  - Kura version KURA_1.4.0 is starting
    11:57:21,961 [Component Resolve Thread] INFO  ConfigurationServiceImpl:155  - activate...
    11:57:21,962 [Component Resolve Thread] INFO  ConfigurationServiceImpl:173  - Trackers being opened...
    ...
    11:57:24,956 [DataServiceImpl:Submit] INFO  DataServiceImpl:552  - DataPublisherService not connected
    11:57:26,953 [] INFO  DataServiceImpl:439  - Storing message on topic :#account-name/#client-id/heater/data, priority: 5
    11:57:26,955 [] INFO  DataServiceImpl:442  - Stored message on topic :#account-name/#client-id/heater/data, priority: 5
    11:57:26,956 [] INFO  Heater:290  - Published to data message: org.eclipse.kura.message.KuraPayload@15b7dbd1
    11:57:26,956 [DataServiceImpl:Submit] INFO  DataServiceImpl:552  - DataPublisherService not connected

(Some output was omitted.)

If you open a web browser to http://localhost:8080/kura, you should be prompted for a username and password. Use `admin` and `admin`, and the Kura web UI should load.

![Kura Web UI](images/webui.png)

Great! The emulator is working! Go ahead and stop the emulator by clicking the red stop button in the Eclipse Console tab.

## Create an App

Now we can create a new project in Eclipse. This project will be our sample sensor application that reads values from sensors and publishes the values to SensorThings. As this is a demonstration, we will generate fake data to simulate a sensor. After the basic application is working, it can be extended to query actual sensors connected to the device running Kura.

In Eclipse, create a new Project from **File | New | Project…**. Select **Plug-in Development > Plug-in Project**, then **Next**.

![New Project Wizard](images/newproject1.png)

For the project name, use `com.example.kura.sensorthings.sampler`. (You can change this, but you will need to change the reference multiple times later.) For the Target Platform, select **an OSGi framework: Standard**. Select **Next**.

![New Project Details](images/newproject2.png)

Uncheck **Generate an activator, a Java class that controls the plug-in’s life cycle** and leave the rest of values at their defaults. Make sure the Execution Environment is Java 8. Select **Finish**. Eclipse will prompt you about "Open Associated Perspective?"; go ahead and select **Yes**.

![Final Project Details](images/newproject3.png)

The new project will now be open in Eclipse, and the `MANIFEST.MF` file open in the editor. In the "Overview" tab, scroll down to "Execution Environments" and remove all the entries. If you have JavaSE-1.8 listed here, then Equinox 3.8.1 will refuse to load it in Kura. This may be fixed in the future with a newer release of Equinox.

Select the "Dependencies" tab in the editor, and expand "Automated Management of Dependencies". Here we will specify that this project will be linked to OSGi, so we can deploy to Kura. Click the **Add…** button. In the window that pops up, filter with `org.eclipse.osgi.services` — a single item should be listed. Select the item and then **OK**. Click **Add…** again and add the `slf4j.api` dependency, so we can do easy logging for debugging later. Save the manifest file.

Now we will create a Java class that will hold the logic of our Kura application. It will handle the configuration, activation/deactivation events, fake sensor data generation, and publishing to SensorThings. To keep things simple these will all be contained in this one class, but later on it would be a good idea to separate some of the functionality to separate classes (see [Single Responsibility Principle](https://en.wikipedia.org/wiki/Single_responsibility_principle)).

In Eclipse, select **File | New | Class**. Set the Source folder to `com.example.kura.sensorthings.sampler/src`. For package, `com.example.kura.sensorthings.sampler`. For Name, `Sampler`. Select **Finish**.

![New Class](images/newclass.png)

The new class will open in the editor. Paste in this sample code:

    package com.example.kura.sensorthings.sampler;

    public class Sampler {

        private static final Logger s_logger = LoggerFactory.getLogger(Sampler.class);
        private static final String APP_ID = "com.example.kura.sensorthings.sampler";

        protected void activate(ComponentContext componentContext) {
            s_logger.info("Bundle " + APP_ID + " has started!");
            s_logger.debug(APP_ID + ": This is a debug message.");
        }

        protected void deactivate(ComponentContext componentContext) {
            s_logger.info("Bundle " + APP_ID + " has stopped!");
        }

    }

Kura will use event hooks to trigger certain methods in our application. Our `activate` method will be triggered when Kura loads the application, and `deactivate` when it is unloaded or removed.

Eclipse will complain about errors in the class we just created. We will fix that by importing the necessary classes for our logger. Select **Source | Organize Imports**, and select `org.slf4j.Logger` in the list, then **Finish**. Save the now updated class file, and the errors will be cleared.

Next we will specify the packages our plugin will need. Open the `META-INF/MANIFEST.MF` file, and select the "Dependencies" tab. Expand the "Automated Management of Dependencies" section and click "add dependencies". Two files will be added to the Imported Packages list. Save the manifest file.

## Configure the Kura Connections

Now we will create the component definition. This will allow our plugin to be loaded with OSGi. Select **File | New | Other**. Select **Plug-in Development > Component Definition**. (If the item is not listed, then you are not using Eclipse IDE for Java EE Developers.) Select **Next**.

For the parent folder, use `com.example.kura.sensorthings.sampler/OSGI-INF`. In the Component Definition Information, change the Name to `com.example.kura.sensorthings.sampler`. Click **Browse…** and type `Sampler` to filter the list to our recently created class. Click **OK**, then **Finish**.

The new component definition file will be opened.

![New Component](images/newcomponent.png)

Change the Activate field to `activate`, and the Deactivate field to `deactivate`. These are the event hooks. Uncheck and re-check "This component is enabled when started", then save the file. Now double-check the values, as the editor is flaky and the values might get reset.

## Test in Emulator

Run the emulator for your platform. In the console log near the top, it should include:

    [Component Resolve Thread] INFO  Sampler:13  - Bundle com.example.kura.sensorthings.sampler has started!

Great! That means the plugin is loaded by Kura. In the web interface under "Device", select the "Bundles" tab. It should include a listing for "Sampler", our plugin.

The plugin doesn't *do* anything right now, but it will soon.

## Create the SensorThings Entities

Our plugin is going to be simple and only will POST observation entities to SensorThings. This means we will need the other entities already present on our SensorThings server. For this project we will create a set of dummy entities. We will use the SensorThings Deep Insert API to create all the required entities in one request.

This example will use [Postman](http://www.getpostman.com); an example with curl follows.

Start a new POST request to your SensorThing instance's `Things` collection, e.g. http://example.com/OGCSensorThings/v1.0/Things

Select the Headers tab and add the key `Content-Type` with the value `application/json`. This is case-sensitive.

Select the Body tab and select "raw". Paste in the following contents:

    {
        "description": "Emulated Kura Device",
        "Locations": [{
            "description": "Emulated Location",
            "encodingType": "application/vnd.geo+json",
            "location": {
                "geometry": {
                    "coordinates": [-114.1337777, 51.080468],
                    "type": "Point"
                }
            },
            "type": "Feature",
            "properties": {}
        }],
        "Datastreams": [{
            "description": "Emulated Light Exposure",
            "observationType": "http://www.opengis.net/def/observationType/OGC-OM/2.0/OM_Observation",
            "unitOfMeasurement": {
                "symbol": "lx",
                "name": "Lux",
                "definition": "http://dbpedia.org/page/Lux"
            },
            "Sensor": {
                "description": "Emulated Lux Sensor",
                "encodingType": "http://schema.org/description",
                "metadata": "Emulated Lux Sensor"
            },
            "ObservedProperty": {
                "name": "Illuminance",
               "definition": "http://dbpedia.org/page/Illuminance",
               "description": "Illuminance is the total luminous flux incident on a surface, per unit area."
            }
        }]
    }

Then click "Send". You should get a similar response with different IDs:

    {
      "@iot.selfLink": "http://example.com/OGCSensorThings/v1.0/Things(240958)",
      "Datastreams@iot.navigationLink": "../Things(240958)/Datastreams",
      "@iot.id": 240958,
      "description": "Emulated Kura Device",
      "Locations@iot.navigationLink": "../Things(240958)/Locations",
      "properties": {},
      "HistoricalLocations@iot.navigationLink": "../Things(240958)/HistoricalLocations"
    }

Do a GET request for the Datastreams, as we will need the first Datastream's Observations navigationLink for our app:

    {
      "@iot.count": 1,
      "value": [
        {
          "@iot.id": 240959,
          "@iot.selfLink": "http://example.com/OGCSensorThings/v1.0/Datastreams(240959)",
          "description": "Emulated Light Exposure",
          "observationType": "http://www.opengis.net/def/observationType/OGC-OM/2.0/OM_Observation",
          "unitOfMeasurement": {
            "symbol": "lx",
            "name": "Lux",
            "definition": "http://dbpedia.org/page/Lux"
          },
          "Observations@iot.navigationLink": "http://example.com/OGCSensorThings/v1.0/Datastreams(240959)/Observations",
          "ObservedProperty@iot.navigationLink": "http://example.com/OGCSensorThings/v1.0/Datastreams(240959)/ObservedProperty",
          "Sensor@iot.navigationLink": "http://example.com/OGCSensorThings/v1.0/Datastreams(240959)/Sensor",
          "Thing@iot.navigationLink": "http://example.com/OGCSensorThings/v1.0/Datastreams(240959)/Thing"
        }
      ]
    }

Note down the URL `http://example.com/OGCSensorThings/v1.0/Datastreams(240959)/Observations`.


### Curl Alternative

Curl is also simple. Create a JSON file named `entities.json` with the entity tree above, and post it to your SensorThings instance:

    $ curl -XPOST http://example.com/OGCSensorThings/v1.0/Things -H "Content-Type: application/json" -d @entities.json

The response will contain the created Thing entity. Do another request for the Thing's Datastreams collection:

    $ curl http://example.com/OGCSensorThings/v1.0/Things(240958)/Datastreams

In the response, note down the first Datastream's Observations collection URL: `http://example.com/OGCSensorThings/v1.0/Datastreams(240959)/Observations`.

## Generate the Sensor Data

Next we will update our Sampler class to run a worker process to simulate sensor data and publish it to SensorThings.


    package com.example.kura.sensorthings.sampler;

    import java.io.DataOutputStream;
    import java.io.IOException;
    import java.net.HttpURLConnection;
    import java.net.MalformedURLException;
    import java.net.URL;
    import java.time.ZonedDateTime;
    import java.time.format.DateTimeFormatter;
    import java.util.concurrent.Executors;
    import java.util.concurrent.ScheduledExecutorService;
    import java.util.concurrent.ScheduledFuture;
    import java.util.concurrent.TimeUnit;

    import org.osgi.service.component.ComponentContext;
    import org.slf4j.Logger;
    import org.slf4j.LoggerFactory;

    public class Sampler {

       	private static final Logger s_logger = LoggerFactory.getLogger(Sampler.class);
       	private static final String APP_ID = "com.example.kura.sensorthings.sampler";

       	private ScheduledExecutorService m_worker;
       	private ScheduledFuture<?> m_handle;

       	public Sampler() {
       		super();
       		m_worker = Executors.newSingleThreadScheduledExecutor();
       	}

       	protected void activate(ComponentContext componentContext) {
       		s_logger.info("Bundle " + APP_ID + " has started!");

       		// cancel a current worker handle if one if active
       		if (m_handle != null) {
       			m_handle.cancel(true);
       		}

       		// schedule a new worker
       		m_handle = m_worker.scheduleAtFixedRate(new Runnable() {
       			@Override
       			public void run() {
       				Thread.currentThread().setName(getClass().getSimpleName());
       				doPublish();
       			}
       		}, 0, 300, TimeUnit.SECONDS);
       	}

       	protected void deactivate(ComponentContext componentContext) {
       		s_logger.info("Bundle " + APP_ID + " has stopped!");
       	}

       	/**
       	 * Called at the configured rate to publish the next temperature
       	 * measurement.
       	 */
       	private void doPublish() {
       		s_logger.info(String.format("Sampler: Preparing a Publish Event"));

       		String observationsURI = "http://example.com/OGCSensorThings/v1.0/Datastreams(240959)/Observations";
       		ZonedDateTime now = ZonedDateTime.now();

       		// Query the sensor value. As it is simulated, we pass in the current
       		// date.
       		float lux = getLux(now);

       		// Get phenomenonTime as string for the Observations
       		String phenomenonTime = now.format(DateTimeFormatter.ISO_INSTANT);

       		// Create Observation
       		createObservation(phenomenonTime, lux, observationsURI);
       	}

       	// Wrap the post call with error handling.
       	private void createObservation(String phenomenonTime, float result, String resourceURI) {
       		try {
       			postObservation(phenomenonTime, result, resourceURI);
       		} catch (MalformedURLException e) {
       			s_logger.warn(String.format("Sampler: Cannot create Observation, as URL is malformed: %s", resourceURI));
       		} catch (IOException e) {
       			s_logger.warn(String.format("Sampler: Cannot create Observation, as a connection could not be opened."));
       		}
       	}

       	// Returns a simulated Lux value for a given time of day.
       	private float getLux(ZonedDateTime instant) {
       		float hour = (float) instant.getHour();
       		float minute = (float) instant.getMinute();
       		float lux = (float) Math.sin((1.5 * Math.PI) + ((Math.PI * hour + (minute / 60)) / 12));
       		return lux;
       	}

       	// Create a new SensorThings Observation with phenomenonTime and value, in
       	// the Observations Collection for the resource at resourceURI. The
       	// resource is most likely a URL to a Datastream.
       	private void postObservation(String phenomenonTime, float result, String resourceURI)
       			throws MalformedURLException, IOException {
       		String postBody = String.format("{\"phenomenonTime\": \"%s\", \"result\": %f}", phenomenonTime, result);

       		URL resourceURL = new URL(resourceURI);
       		HttpURLConnection connection = (HttpURLConnection) resourceURL.openConnection();

       		connection.setDoOutput(true);
       		connection.setInstanceFollowRedirects(false);
       		connection.setRequestMethod("POST");
       		connection.setRequestProperty("Content-Type", "application/json");

       		DataOutputStream writer = new DataOutputStream(connection.getOutputStream());
       		writer.writeBytes(postBody);
       		writer.flush();
       		writer.close();

       		int responseCode = connection.getResponseCode();
       		s_logger.info(String.format("Sampler Response Code: %d", responseCode));
       	}

    }

There is quite a bit of new stuff. In our `activate()` function, we now create a worker thread that every five minutes will trigger the `doPublish()` function. There, we generate an observation result then create a POST request to SensorThings. Be sure to edit the `observationsURI` to point to your collection from the previous step. Save the class.

## Test in Emulator

To clear up the clutter in the emulator console, disabled the demo heater project. You can do that by right-clicking on the org.eclipse.kura.demo.heater project and selecting "Close Project".

Now if you run the emulator, you should see logs from our worker process (unrelated lines omitted):

    14:58:05,426 [Component Resolve Thread] INFO  Sampler:34  - Bundle com.example.kura.sensorthings.sampler has started!
    14:58:05,428 [] INFO  Sampler:59  - Sampler: Preparing a Publish Event
    14:58:05,622 [] INFO  Sampler:112  - Sampler Response Code: 201
    15:03:05,422 [] INFO  Sampler:59  - Sampler: Preparing a Publish Event
    15:03:05,642 [] INFO  Sampler:112  - Sampler Response Code: 201

Excellent! If we load the Observations collection, we should see the new Observations:

    {
      "@iot.count": 3,
      "value": [
        {
          "@iot.id": 241205,
          "@iot.selfLink": "http://example.com/OGCSensorThings/v1.0/Observations(241205)",
          "phenomenonTime": "2016-04-18T21:03:05.423Z",
          "result": "0.704154",
          "resultTime": null,
          "Datastream@iot.navigationLink": "http://example.com/OGCSensorThings/v1.0/Observations(241205)/Datastream",
          "FeatureOfInterest@iot.navigationLink": "http://example.com/OGCSensorThings/v1.0/Observations(241205)/FeatureOfInterest"
        },
        {
          "@iot.id": 241167,
          "@iot.selfLink": "http://example.com/OGCSensorThings/v1.0/Observations(241167)",
          "phenomenonTime": "2016-04-18T20:58:05.437Z",
          "result": "0.822983",
          "resultTime": null,
          "Datastream@iot.navigationLink": "http://example.com/OGCSensorThings/v1.0/Observations(241167)/Datastream",
          "FeatureOfInterest@iot.navigationLink": "http://example.com/OGCSensorThings/v1.0/Observations(241167)/FeatureOfInterest"
        },
        {
          "@iot.id": 241155,
          "@iot.selfLink": "http://example.com/OGCSensorThings/v1.0/Observations(241155)",
          "phenomenonTime": "2016-04-18T20:56:14.228Z",
          "result": "0.824558",
          "resultTime": null,
          "Datastream@iot.navigationLink": "http://example.com/OGCSensorThings/v1.0/Observations(241155)/Datastream",
          "FeatureOfInterest@iot.navigationLink": "http://example.com/OGCSensorThings/v1.0/Observations(241155)/FeatureOfInterest"
        }
      ]
    }

This will continue to run until the emulator is stopped. If you goto the emulator's web UI at http://127.0.0.1:8080/kura, select "Device" then "Bundles", you can stop the Sampler app by selecting it and clicking the stop button. The emulator log will show:

    15:07:44,304 [qtp2024913552-25] INFO  Sampler:52  - Bundle com.example.kura.sensorthings.sampler has stopped!

And if you re-enable it:

    15:09:03,457 [Component Resolve Thread (Bundle 50)] INFO  Sampler:34  - Bundle com.example.kura.sensorthings.sampler has started!
    15:09:03,458 [] INFO  Sampler:59  - Sampler: Preparing a Publish Event
    15:09:03,553 [] INFO  Sampler:112  - Sampler Response Code: 201

Go ahead and stop the emulator.

## Conclusion

The code required to run a small sensor application on Kura that publishes to SensorThings is simple and easy to get started. With this tutorial, I hope you can build and deploy your own SensorThings application for Kura and connect something new!

## Source Code

This tutorial is available in a git repository at [Github](https://github.com/GeoSensorWebLab/kura-sensorthings-tutorial) or [Bitbucket](https://bitbucket.org/geosensorweblab/kura-sensorthings-tutorial). The source code for the Sampler app is also available on [Github](https://github.com/GeoSensorWebLab/com.example.kura.sensorthings.sampler) or [Bitbucket](https://bitbucket.org/geosensorweblab/com.example.kura.sensorthings.sampler).

## Author

James Badger <jpbadger@ucalgary.ca>

## License

This tutorial is [CC-BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/). Code examples are under MIT License.
