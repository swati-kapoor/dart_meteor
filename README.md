For VM, Flutter iOS/Android (master branch) ![](https://github.com/tanutapi/dart_meteor/workflows/Testing/badge.svg?branch=master)

For Flutter Web (web branch) ![](https://github.com/tanutapi/dart_meteor/workflows/Testing/badge.svg?branch=web)

# A Meteor DDP library for Dart/Flutter developers.

This library makes a connection between the Meteor backend and the Flutter app simply. Design to work seamlessly with StreamBuilder and FutureBuilder.

## Usage

I just publish a post on Medium showing how to handle connection status, user authentication, and subscriptions. Please check https://medium.com/@tanutapi/writing-flutter-mobile-application-with-meteor-backend-643d2c1947d0?source=friends_link&sk=52ce2fa2603934e7395e2d19dd54e06c

A simple usage example:

First, create an instance of MeteorClient in your app global scope so that it can be used anywhere in your project.

```dart
import 'package:flutter/material.dart';
import 'package:dart_meteor/dart_meteor.dart';

MeteorClient meteor = MeteorClient.connect(url: 'https://yourdomain.com');
void main() => runApp(MyApp());
```

In your StatefulWidget/StatelessWidget, thanks to [rxdart][rxdart], you can use FutuerBuilder or StreamBuilder to build your widget base on a response from meteor's DDP server.

```dart
class MyApp extends StatefulWidget {
  @override
  _MyAppState createState() => _MyAppState();
}

class _MyAppState extends State<MyApp> {
  String _methodResult = '';

  void _callMethod() {
    meteor.call('helloMethod').then((result) {
      setState(() {
        _methodResult = result.toString();
      });
    }).catchError((err) {
      if (err is MeteorError) {
        setState(() {
          _methodResult = err.message;
        });
      }
    });
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(
          title: Text('Package dart_meteor Example'),
        ),
        body: Container(
          padding: EdgeInsets.all(8.0),
          child: Column(
            children: <Widget>[
              StreamBuilder<DdpConnectionStatus>(
                stream: meteor.status(),
                builder: (context, snapshot) {
                  if (snapshot.hasData) {
                    if (snapshot.data.status ==
                        DdpConnectionStatusValues.connected) {
                      return RaisedButton(
                        child: Text('Disconnect'),
                        onPressed: () {
                          meteor.disconnect();
                        },
                      );
                    }
                    return RaisedButton(
                      child: Text('Connect'),
                      onPressed: () {
                        meteor.reconnect();
                      },
                    );
                  }
                  return Container();
                },
              ),
              StreamBuilder<DdpConnectionStatus>(
                stream: meteor.status(),
                builder: (context, snapshot) {
                  if (snapshot.hasData) {
                    return Text('Meteor Status ${snapshot.data.toString()}');
                  }
                  return Text('Meteor Status: ---');
                },
              ),
              StreamBuilder(
                  stream: meteor.userId(),
                  builder: (context, snapshot) {
                    if (snapshot.hasData) {
                      return RaisedButton(
                        child: Text('Logout'),
                        onPressed: () {
                          meteor.logout();
                        },
                      );
                    }
                    return RaisedButton(
                      child: Text('Login'),
                      onPressed: () {
                        meteor.loginWithPassword(
                            'yourusername', 'yourpassword');
                      },
                    );
                  }),
              StreamBuilder(
                stream: meteor.user(),
                builder: (context, snapshot) {
                  if (snapshot.hasData) {
                    return Text(snapshot.data.toString());
                  }
                  return Text('User: ----');
                },
              ),
              RaisedButton(
                child: Text('Method Call'),
                onPressed: _callMethod,
              ),
              Text(_methodResult),
            ],
          ),
        ),
      ),
    );
  }
}
```

## Making a method call to your server

Making a method call to your server returns a Future. You MUST handle `catchError` to prevent your app from crashing if something went wrong. 

```dart
meteor.call('helloMethod').then((result) {
  setState(() {
    _methodResult = result.toString();
  });
}).catchError((err) {
  if (err is MeteorError) {
    setState(() {
      _methodResult = err.message;
    });
  }
});
```
You can also use it with a FutureBuilder.
```dart
FutureBuilder<int>(
  future: meteor.call('sumMethod', args: [5, 10]),
  builder: (context, snapshot) {
    if (snapshot.hasData) {
      // your snapshot.data should = 5 + 10 = 15
    }
  },
),
```

You can found an example project inside [/example][example].

## Collections & Subscriptions
You can access your collections by calling `collection('your_collection_name')`.
It will return a `Stream` which you can use it with your `StreamBuilder` or wherever you want. Through the returned `Stream` reference you can listen the updates of the collection.

```dart
meteor.collection('your_collections');
```

Which return a stream that backed by rxdart BehaviorSubject, a special StreamController that captures the latest item that has been added to the controller, and emits that as the first item to any new listener. You can use it as a simple Stream. To make collections available in Flutter app you might make a subscription to your server with:

```dart
class YourWidget extends StatefulWidget {
  YourWidget() {}

  @override
  _YourWidgetState createState() => _YourWidgetState();
}

class _YourWidgetState extends State<YourWidget> {
  SubscriptionHandler _subscriptionHandler;

  @override
  void initState() {
    super.initState();
    _subscriptionHandler = meteor.subscribe('your_pub', args: ['param_1', 'param_2']);
  }

  @override
  void dispose() {
    super.dispose();
    _subscriptionHandler.stop();
  }

  @override
  Widget build(BuildContext context) {
    return StreamBuilder(
      stream: meteor.collection('your_collection'),
      builder:
          (context, AsyncSnapshot<Map<String, dynamic>> snapshot) {
        int docCount = 0;
        if (snapshot.hasData) {
          docCount = snapshot.data.length;
        }
        return Text('Total document count: $docCount');
      },
    );
  }
}
```

The collection was return as a Map<String, dynamic>. The key is a document .\_id and its value is the whole document.

Ex.
```
{
  "DGbsysgxzSf7Cr8Jg": {
    "_id": "DGbsysgxzSf7Cr8Jg", 
    field1: 0, 
    field2: "a", 
    field3: true, 
    field4: SomeDate
  }
}
```
We did not provide some kind of minimongo. We believe that you can use `reduce`, `map`, and `where` with the collection and get the same result as you did with a query in `minimongo` `meteor` web client.

## Features and bugs

Please file feature requests and bugs at the [issue tracker][tracker].

[tracker]: https://github.com/tanutapi/dart_meteor/issues
[rxdart]: https://pub.dev/packages/rxdart
[example]: https://github.com/tanutapi/dart_meteor/tree/master/example
