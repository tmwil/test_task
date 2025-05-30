How to Create a Custom WebView for MyCap Data

Creating a custom WebView for MyCap Data allows your website to send structured JSON data back to the Flutter app via JavaScript. This guide will walk you through how to set up your website to properly communicate with the app, ensuring seamless integration.
1. Understanding How the WebView Receives Data

The Flutter app is built using webview_flutter, and it listens for JavaScript messages sent via a JavaScript Channel named "returnData". Your website needs to include JavaScript logic to send data using window.returnData.postMessage(...).

Your website must:

    Be publicly accessible on the web.
    Have JavaScript enabled.
    Format responses as JSON strings.

2. Setting Up Your Website to Send JSON Data

You can use any backend language (Node.js, Python, PHP, etc.) to generate and serve data. However, the key part is your frontend JavaScript that communicates with the WebView.
Example Web Page (HTML + JavaScript)

This simple page allows users to enter data, which is then sent to the Flutter WebView.

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>WebView Input</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            height: 100vh;
            text-align: center;
        }
        input {
            padding: 10px;
            width: 80%;
            max-width: 300px;
            margin-bottom: 10px;
        }
        button {
            padding: 10px;
            cursor: pointer;
        }
        .error {
            color: red;
            margin-top: 10px;
        }
    </style>
</head>
<body>

    <h2>Enter Text</h2>
    <input type="text" id="userInput" placeholder="Type something...">
    <button onclick="submitData()">Submit</button>
    <p id="errorMessage" class="error" style="display: none;"></p>

    <script>
    function submitData() {
        const inputValue = document.getElementById("userInput").value;
        const result = JSON.stringify({ text: inputValue });
        const errorMessage = document.getElementById("errorMessage");

        try {
            if (window.returnData && typeof window.returnData.postMessage === "function") {
                window.returnData.postMessage(result);
                console.log("Data sent successfully:", result);
            } else {
                throw new Error("Flutter JavaScript channel 'returnData' is not available.");
            }

            // Close WebView after sending data
            setTimeout(() => {
                window.close();
            }, 500);
        } catch (error) {
            console.error("Error sending data:", error);
            errorMessage.textContent = error.message;
            errorMessage.style.display = "block"; // Show error message on the screen
        }
    }
    </script>

</body>
</html>

How This Works

    The user enters data into the text box.
    Clicking the Submit button converts it into a JSON object:

{ "text": "User input here" }

The JSON is sent to the Flutter app using:

    window.returnData.postMessage(result);

    If the WebView in Flutter is active and listening, it will receive this data and update the UI accordingly.

3. How to Host Your Website

Your webpage must be hosted online for the Flutter WebView to access it. Here are a few hosting options:
Hosting Service	Notes
Netlify	Free tier available, simple to use
Vercel	Great for JavaScript-based sites
GitHub Pages	Works well for static sites
Firebase Hosting	Google-backed hosting
Custom Server	Use Nginx/Apache to serve

Once hosted, your page will have a URL like:

https://your-site.com/webview-page.html

This URL should be embedded in the Flutter WebView so the app can load it.
4. Backend Example (Optional)

If your site needs to dynamically generate JSON responses, you can create an API endpoint. Below are examples in different backend languages.
Node.js (Express) API

const express = require('express');
const app = express();

app.get('/data', (req, res) => {
    res.json({ text: "Hello from the server!" });
});

app.listen(3000, () => console.log("Server running on port 3000"));

Python (Flask) API

from flask import Flask, jsonify

app = Flask(__name__)

@app.route('/data', methods=['GET'])
def get_data():
    return jsonify({"text": "Hello from the Python server!"})

if __name__ == '__main__':
    app.run(debug=True)

If using a backend, your JavaScript should fetch this API and send the data to Flutter:

fetch('https://your-api.com/data')
    .then(response => response.json())
    .then(data => window.returnData.postMessage(JSON.stringify(data)));

5. Integrating with Flutter WebView

In the Flutter app, ensure the WebView is correctly configured to listen for messages from the web page.
Flutter Code

import 'package:flutter/material.dart';
import 'package:webview_flutter/webview_flutter.dart';

void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Flutter WebView Demo',
      theme: ThemeData(primarySwatch: Colors.blue),
      home: WebViewScreen(),
    );
  }
}

class WebViewScreen extends StatefulWidget {
  @override
  _WebViewScreenState createState() => _WebViewScreenState();
}

class _WebViewScreenState extends State<WebViewScreen> {
  WebViewController? _webViewController;
  String receivedData = "No data received yet";

  void _startController() {
    setState(() {
      _webViewController = WebViewController()
        ..setJavaScriptMode(JavaScriptMode.unrestricted)
        ..addJavaScriptChannel(
          'returnData', // Matches JavaScript channel
          onMessageReceived: (message) {
            setState(() {
              receivedData = message.message;
            });
          },
        )
        ..loadRequest(Uri.parse("https://your-site.com/webview-page.html"));
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('WebView Example')),
      body: receivedData == "No data received yet"
          ? _webViewController != null
              ? WebViewWidget(controller: _webViewController!)
              : Center(
                  child: ElevatedButton(
                    onPressed: _startController,
                    child: const Text("Start WebView"),
                  ),
                )
          : Center(
              child: Text(
                receivedData,
                style: const TextStyle(fontSize: 18),
              ),
            ),
    );
  }
}

6. Summary

✅ Your website must:

    Host a web page with JavaScript that sends JSON data to window.returnData.postMessage(...).
    Be publicly accessible (Netlify, Vercel, Firebase, etc.).
    Optionally, have a backend API to generate data dynamically.

✅ Your Flutter app must:

    Load the web page in WebViewController().
    Listen for messages via JavaScript Channels.
    Process received JSON data.

With this setup, MyCap Data can receive structured data from any website hosted on the web, regardless of the backend technology used.

🚀 Now you can integrate any external web service with Flutter WebView effortlessly!
