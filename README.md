##Firebase Web Push Notification
install all packages from <b>package.json</b> file

```npm install```

index.js file
```js
require('dotenv')
var express = require('express');
var app = express();

app.use(express.static('public'));

app.get('/', function (req, res) {
	res.sendFile( __dirname + "public" + "index.html" );
})

var server = app.listen(3000, function () {
	var host = server.address().address
	var port = server.address().port
	console.log("App listening at http://%s:%s", host, port)
})
```
Now create firebase-messaging-sw.js at the public folder. The file name should be same because this is the convention for service worker according to Firebase. 
I hope you know about service worker if not please read [https://developers.google.com/web/fundamentals/primers/service-workers/](this). 
Paste below code into the service worker js file.

```js
/*
Give the service worker access to Firebase Messaging.
Note that you can only use Firebase Messaging here, other Firebase libraries are not available in the service worker.
*/
importScripts('https://www.gstatic.com/firebasejs/4.13.0/firebase-app.js')
importScripts('https://www.gstatic.com/firebasejs/4.13.0/firebase-messaging.js')

/*
Initialize the Firebase app in the service worker by passing in the messagingSenderId.
*/
firebase.initializeApp({
    //'messagingSenderId': process.env.MESSAGING_SENDER_ID
    'messagingSenderId': 'SENDER_ID'
})

/*
Retrieve an instance of Firebase Messaging so that it can handle background messages.
*/
const messaging = firebase.messaging()
messaging.setBackgroundMessageHandler(function (payload) {
    console.log('[firebase-messaging-sw.js] Received background message ', payload);
    const notification = JSON.parse(payload.data.notification);
    // Customize notification here
    const notificationTitle = notification.title;
    const notificationOptions = {
        body: notification.body,
        icon: notification.icon
    };

    return self.registration.showNotification(notificationTitle,
        notificationOptions);
});

```

Now create index.html where our web page and user token capture code is there. Use below code for index.html.

```html
<!DOCTYPE html>
<html>

<head>
	<script src="https://www.gstatic.com/firebasejs/4.13.0/firebase-app.js"></script>
	<script src="https://www.gstatic.com/firebasejs/4.13.0/firebase-messaging.js"></script>
	<script>
		firebase.initializeApp({
			'messagingSenderId': 'SENDER_ID'
		})
		const messaging = firebase.messaging();
		function initFirebaseMessagingRegistration() {
			messaging
					.requestPermission()
					.then(function () {
						messageElement.innerHTML = "Got notification permission";
						console.log("Got notification permission");
						return messaging.getToken();
					})
					.then(function (token) {
						// print the token on the HTML page
						tokenElement.innerHTML = "Token is " + token;
					})
					.catch(function (err) {
						errorElement.innerHTML = "Error: " + err;
						console.log("Didn't get notification permission", err);
					});
		}
		messaging.onMessage(function (payload) {
			console.log("Message received. ", JSON.stringify(payload));
			notificationElement.innerHTML = notificationElement.innerHTML + " " + payload.data.notification;
		});
		messaging.onTokenRefresh(function () {
			messaging.getToken()
					.then(function (refreshedToken) {
						console.log('Token refreshed.');
						tokenElement.innerHTML = "Token is " + refreshedToken;
					}).catch(function (err) {
				errorElement.innerHTML = "Error: " + err;
				console.log('Unable to retrieve refreshed token ', err);
			});
		});
	</script>
</head>

<body>
<h1>This is a test page</h1>
<div id="token" style="color:lightblue"></div>
<div id="message" style="color:lightblue"></div>
<div id="notification" style="color:green"></div>
<div id="error" style="color:red"></div>
<script>
	messageElement = document.getElementById("message")
	tokenElement = document.getElementById("token")
	notificationElement = document.getElementById("notification")
	errorElement = document.getElementById("error")
</script>
<button onclick="initFirebaseMessagingRegistration()">Enable Firebase Messaging</button>

</html>
```

Now to send the push notification to your customer(for now it’s you) run this below command after replacing with your data.

```
scurl -X POST -H "Authorization: key=SERVER_KEY" -H "Content-Type: application/json"    -d '{
      "data": {
        "notification": {
            "title": "FCM Message",
            "body": "This is an FCM Message",
            "icon": "/ab-logo.png",
        }
      },
      "to": "USER_TOKEN"
    }' https://fcm.googleapis.com/fcm/send
```

Replace SERVER_KEY with the server key you got in Firebase project, Replace USER_TOKEN with the user token which you got on page after allowing show notifications. If possible replace the icon with the icon name you put in your code public folder.
