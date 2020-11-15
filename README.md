# Diving into the Capacitor Community HTTP Plugin

## What is the Capacitor Community HTTP Plugin?

The [HTTP plugin](https://github.com/capacitor-community/http) from the Capacitor Community is a versatile toolkit for native HTTP requests, file uploads/downloads, and cookie management.

By using native HTTP requests, developers prevent the dreaded, frustrating CORS issues that can happen when making requests from a browser. The HTTP plugin makes this easy by providing functions that make the request natively instead.

## What we’ll learn

In this tutorial, we’ll be demonstrating two main features of the HTTP plugin:

1. Native HTTP Requests (GET)
2. File uploads

This tutorial is divided into four main sections.

1. An overview of the functionality and structure of our demo application
2. Building the native HTTP request functionality using the plugin
3. Building the file upload functionality using the plugin
4. Building the native application with Capacitor

The application and API code are open-sourced on GitHub:

- [ballot-cure repository](https://github.com/ceceliacreates/ballot-cure)
- [ballot-api repository](https://github.com/ceceliacreates/ballot-api)

## What we’re building

We’ll be building a ballot curing app using the new Ionic Vue. Ballot curing became a hot issue in the recent U.S. election. When a problem occurs with an absentee ballot, voters must “cure” or resolve the issue by a deadline for their vote to count.

The functionality is similar to any system that requires a user to submit documents for verification. These are typical in financial, education, healthcare, and government applications where users may need to submit legal documents like a driver’s license or power of attorney.

> Note: If you like the idea of contributing to open-source projects that support government and civic programs, check out [Code for America](https://www.codeforamerica.org/).

### App functionality

Our application provides a simple interface for the following functionality:

1. Search for a ballot by ID
2. See results of ballot search, including the issue that needs to be resolved
3. Allows user to upload a file to resolve the issue
4. Submits the file for the corresponding ballot ID

![video of app functionality](https://raw.githubusercontent.com/ceceliacreates/ballot-cure/main/BallotCureDemo.gif)

## Building the ballot search with native HTTP requests

### Installation and setup

We’ll use the Ionic CLI to create our Vue app with the blank starter, following the [Ionic Vue Quickstart](https://ionicframework.com/docs/vue/quickstart#creating-a-project-with-the-ionic-cli). This will also set up Capacitor so we can build to Android and iOS after building our app.

```
ionic start ballot-cure blank --type vue
```

Then we cd into our project directory and run `ionic serve` to start up the development server for our app.

### Building the search user interface

To focus on the functionality of the HTTP plugin, we’ll keep the UI simple with a single view. We can leverage components from the [Ionic Framework UI library](https://ionicframework.com/docs/components) to build our view quickly.

Let’s start with the part of the application where the user searches for a ballot by ID. In our `Home.vue`, we will delete the content in the container `div` and replace it with text instructions, an input field, and a search button. We are using `v-model` to bind the input field to the `searchInput` data field.

```javascript
<div class="ion-padding">
<ion-text color="primary">
    <h2>Instructions</h2>
</ion-text>
<p>Enter the ballot ID provided in your notice.</p>
</div>
<ion-item>
    <ion-label>Enter Ballot ID</ion-label>
    <ion-input v-model="searchInput" type="text"></ion-input>
    <ion-button @click="search">Search</ion-button>
</ion-item>
```

We’ll also need an area to display the results of the ballot search. In an `ion-card` component, we’ll display the voter name, address, and ballot issue that are returned. We’ll use `v-if` to only display this component once we have a ballot, and wrap it in a `transition` component to fade in on appearance.

```javascript
<transition name="fade">
    <ion-card v-if="ballot.voterFirstName">
        <ion-card-header>
        <ion-card-title>Ballot Information</ion-card-title>
        <ion-card-subtitle
                >{{ ballot.voterFirstName }}
                {{ ballot.voterLastName }}</ion-card-subtitle
              >
        <ion-card-subtitle>{{ ballot.voterAddress }}</ion-card-subtitle>
        </ion-card-header>
        <ion-card-content>
            <ion-text color="danger"
                ><h2>Issue Requiring Resolution:</h2></ion-text
              >
            <p>{{ ballot.issueDescription }}</p>
        </ion-card-content>
    </ion-card>
</transition>
```

Finally, let’s update the name in the header and add an icon for styling. We’ll need to return our icon in the `setup()` function of our component definition.

```javascript
<ion-header :translucent="true">
    <ion-toolbar>
        <ion-title>
            <ion-icon :icon="shieldCheckmarkOutline" size="large"></ion-icon>BallotCure
        </ion-title>
    </ion-toolbar>
</ion-header>
```

### Building the front-end logic

Now let’s set up the logic for our component. We’ll use the Composition API built into Vue 3 to handle the component setup, data, watchers, and methods. There is an excellent breakdown of the API [in the Vue docs here](https://v3.vuejs.org/guide/composition-api-introduction.html#why-composition-api).

For the search functionality, we’ll need the following:

- Data fields for the `searchInput`, `ballotId`, and `ballot`
- A `search()` method to handle the button click, validate the input, and set the `ballotId`
- A watch to trigger our `getBallot()` method whenever the `ballotId` is updated
- A `getBallot()` method to get the ballot for the given `ballotId` and update our `ballot` data field
- A helper `fetchBallot()` method to handle the API call associated with getting the ballot

This is what our code looks like inside our composition function:

data()

```javascript
data() {
    return {
      searchInput: "",
      ballotId: "",
      ballot: {},
      fileName: "",
    };
  },
```

watch:

```javascript
  watch: {
    ballotId(newBallotId, oldBallotId) {
      this.getBallot(newBallotId);
    },
  },
```

methods:

```javascript
methods: {
    async fetchBallot(id: string) {
      const { Http } = Plugins;
      const response = await Http.request({
        method: "GET",
        url: `https://hungry-brown-da828c.netlify.app/.netlify/functions/server/ballots/${id}`,
      });
      const ballot = response.data;
      return ballot;
    },
    async getBallot(id: string) {
      const newBallot = await this.fetchBallot(id);
      this.ballot = newBallot;
    },
    search() {
      if (this.searchInput.length === 5) {
        const newBallotId = this.searchInput;
        this.ballotId = newBallotId;
      }
    },
    submit() {
      console.log(this.fileName);
    },
  },
```

### Making requests to the API

We’ll be making requests to an Express API deployed as a [Netlify Function](https://docs.netlify.com/functions/overview/). As a reminder, you can see the code for the API [in the repository here](https://github.com/ceceliacreates/ballot-api). The deployed endpoint URL is:

[https://hungry-brown-da828c.netlify.app/.netlify/functions/server/ballots](https://hungry-brown-da828c.netlify.app/.netlify/functions/server/ballots)

Our app will make two requests to a single `/ballots/:id` endpoint:

1. A `GET` that returns the ballot matching the passed ID
2. A `POST` request that updates the ballot matching the passed ID (handled by our `HTTP.fileUpload()` function)

Note: Because we aren’t updating a database, our data is not persistent. This Function is just for demonstration and to process our front-end request.

### Bypassing CORS issues with native HTTP requests

So what happens when we make a request to the endpoint directly from an application running in the browser?

![screenshot of a CORS error](https://raw.githubusercontent.com/ceceliacreates/ballot-cure/main/cors.PNG)

😱

CORS issues are particularly tricky because they normally cannot be resolved with front-end code. There’s [a great breakdown of CORS issues in the Ionic documentation](https://ionicframework.com/docs/troubleshooting/cors), but ultimately CORS issues can only be resolved in one of two ways -- change the API, or don’t make the request from the browser.

In our case, we could use the [Express cors middleware](https://expressjs.com/en/resources/middleware/cors.html) and add code like

```javascript
const cors = require(“cors”)
app.use(cors())
```

to our API to accept **all** requests from the browser, but this is not secure. Configuring our API to receive only the correct browser requests can be complex.

To resolve, let’s use a native request instead with the HTTP plugin.

### Installing and configuring the HTTP Plugin

Following the installation instructions for the plugin, we start by installing the package.

```
npm install @capacitor-community/http
```

Then we can either use the Ionic CLI or npx to sync the plugin with Capacitor.

```
npx cap sync
```

OR

```
ionic cap sync
```

### Usage in our application

When using the plugin for HTTP requests, you’ll need to import the package as well as Plugins from capacitor/core. In our application, we do this right in the script section of our Home view.

```javascript
import "@capacitor-community/http";
import { Plugins } from "@capacitor/core";
```

Then, we destructure the plugin as close to its usage as possible. In our app, we do this in our `fetchBallot()` method that uses `HTTP.request()` to make the native request that returns our ballot information.

```javascript
async fetchBallot(id: string) {
    const { Http } = Plugins;
    const response = await Http.request({
        method: "GET",
        url: `https://hungry-brown-da828c.netlify.app/.netlify/functions/server/ballots/${id}`,
      });
    const ballot = response.data;
    return ballot;
},
```

Now when we search for our ballot, the `GET` request will resolve like magic, because it’s being handled natively by the plugin.

> Note: The ballot-api repository has a version of the API that you can run locally with cors enabled for testing during development. You can access it on the cors-enabled branch.

The same function can be used for `POST` requests as well. For our application, we'll let the HTTP Plugin `fileUpload()` functionality handle the request for us.

## File uploads with the HTTP Plugin

### Adding the file selector and submit button

Let’s add the elements we’ll need to our template. To better style our file selector, we’ll wrap the input in a `div` and use CSS to override the display. We are using `v-model` to bind this to a `fileName` data field. Because we have set a type of "file" on the `ion-input`, clicking the input will automatically open the system's file selector.

```javascript
<div v-if="ballot.voterFirstName">
  <div class="fileUpload ion-padding">
    <span>Click to upload required document</span>
    <ion-input
      type="file"
      name="file"
      v-model="fileName"
      class="upload"
    ></ion-input>
  </div>
</div>
```

We also have a `div` that shows the selected file name (with the fakepath removed) that only appears once a file is selected. Our submit button triggers the `submit()` method on click.

```javascript
<transition name="fade">
    <div v-if="fileName">
        <p>{{ fileName.slice(12) }}</p>
    </div>
</transition>
<ion-button class="ion-margin" @click="submit" size="large">Submit</ion-button>
```

For our logic, we’ll add our data field for `fileName` and write our `submit()` method.

### Using the HTTP.fileUpload() method

[Behind the scenes, the HTTP plugin submits the file as formData to our API](https://github.com/capacitor-community/http/blob/master/src/web.ts#L162). It submits the file uploaded, as well as the file name. You can see the [full list of options available on the fileUpload function here](https://github.com/capacitor-community/http/blob/master/src/definitions.ts#L69).

Inside our `submit()` method, we’ll use the `HTTP.fileUpload()` method and pass the URL of our endpoint as well as the `fileName`. The plugin function will submit the file as a `POST` request to our endpoint.

```javascript
async submit() {
    const { Http } = Plugins;
    const ret = await Http.uploadFile({
        url: `https://hungry-brown-da828c.netlify.app/.netlify/functions/server/ballots/${this.ballotId}`,
        name: this.fileName,
        filePath: this.fileName,
});
},
```

### Handling the uploaded file in our API

In our example, we process this file using an [Express multer middleware.](http://expressjs.com/en/resources/middleware/multer.html) We can then set this file as a value in our ballot.

```javascript
// middleware to handle the file upload
const multer = require("multer");
const upload = multer({});

// Express can access the submitted file on the request body
router.post("/ballots/:id", upload.any(), function(req, res) {
  const ballotId = req.params.id;
  const ballotIndex = ballots.findIndex((ballot) => ballot.id === ballotId);
  const ballotToUpdate = ballots[ballotIndex];
  if (req.body) {
    ballotToUpdate.issueResolutionFile = req.body;
    res.json(ballotToUpdate);
  } else {
    const error = {
      message: "file required",
    };
    res.json(error);
  }
});
```

For additional security, we could require the user enter a PIN that we validate on our API before updating.

Currently, the `HTTP.fileUpload()` function returns an empty object, but in future versions we could use the `POST` request response to confirm the upload was successfule for the user.

## Building our native application

Once we’ve completed the source code for our application, we can build our application for Android and iOS using the [Capacitor setup instructions here in the Ionic documentation](https://ionicframework.com/docs/vue/your-first-app/6-deploying-mobile#capacitor-setup).

First, run `ionic build` and then add the projects.

```
ionic cap add ios
ionic cap add android
```

#### Add to Android Main Activity

For Android, you’ll need to import and register the plugin in your `MainActivity` file. In our application, this is in `android/app/src/main/java/io/ionic/starter/MainActivity.java`.

This file is generated when we build the Ionic app, so ensure you register the plugin after completing and building the source code.

```java
import com.getcapacitor.plugin.http.Http;

public class MainActivity extends BridgeActivity {
@Override
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    // Initializes the Bridge
    this.init(savedInstanceState, new ArrayList&lt;Class&lt;? extends Plugin>>() {{
      // Additional plugins you've installed go here
      // Ex: add(TotallyAwesomePlugin.class);
      add(Http.class);
    }});
}
}
```

You can then open the project and run the application.

## Going further with HTTP Plugin

We’ve outlined in detail how to use native HTTP requests and file uploads with the plugin. In addition, you can also:

1. Download files
2. Manage cookies

If we wanted to demonstrate these with our application, we could add functionality to download a PDF of instructions, or use cookies to save the ballot ID entered by the user for persistence. These would make great open-source contributions to this project! 😉

You can do a lot or a little with the HTTP Plugin, so it’s a good fit for a variety of projects that need native HTTP requests, file upload and downloads, and cookie management.

For more information, check out the [plugin repository here](https://github.com/capacitor-community/http), and the code for [this tutorial application here](https://github.com/ceceliacreates/ballot-cure).
