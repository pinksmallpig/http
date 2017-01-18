# http

A composable, Future-based library for making HTTP requests.

This package contains a set of high-level functions and classes that make it
easy to consume HTTP resources.

**Note:** This is a Flutter-only fork of [the main `http` package][http] that
doesn't provide browser support. It exists as a workaround for [issue 24581][],
which will allow Dart libraries to support multiple platforms at once without
using mirrors. It can only be installed in Flutter environments.

[http]: https://pub.dartlang.org/packages/http
[issue 24581]: https://github.com/dart-lang/sdk/issues/24581

## Using

The easiest way to use this library is via the top-level functions. They allow
you to make individual HTTP requests with minimal hassle:

```dart
import 'package:http/http.dart' as http;

var url = "http://example.com/whatsit/create";
http.post(url, body: {"name": "doodle", "color": "blue"})
    .then((response) {
  print("Response status: ${response.statusCode}");
  print("Response body: ${response.body}");
});

http.read("http://example.com/foobar.txt").then(print);
```

If you're making multiple requests to the same server, you can keep open a
persistent connection by using a [Client][] rather than making one-off requests.
If you do this, make sure to close the client when you're done:

```dart
var client = new http.Client();
client.post(
    "http://example.com/whatsit/create",
    body: {"name": "doodle", "color": "blue"})
  .then((response) => client.get(response.bodyFields['uri']))
  .then((response) => print(response.body))
  .whenComplete(client.close);
```

You can also exert more fine-grained control over your requests and responses by
creating [Request][] or [StreamedRequest][] objects yourself and passing them to
[Client.send][].

[Request]: https://www.dartdocs.org/documentation/http/latest/http/Request-class.html
[StreamedRequest]: https://www.dartdocs.org/documentation/http/latest/http/StreamedRequest-class.html
[Client.send]: https://www.dartdocs.org/documentation/http/latest/http/Client/send.html

This package is designed to be composable. This makes it easy for external
libraries to work with one another to add behavior to it. Libraries wishing to
add behavior should create a subclass of [BaseClient][] that wraps another
[Client][] and adds the desired behavior:

[BaseClient]: https://www.dartdocs.org/documentation/http/latest/http/BaseClient-class.html
[Client]: https://www.dartdocs.org/documentation/http/latest/http/Client-class.html

```dart
class UserAgentClient extends http.BaseClient {
  final String userAgent;
  final http.Client _inner;

  UserAgentClient(this.userAgent, this._inner);

  Future<StreamedResponse> send(BaseRequest request) {
    request.headers['user-agent'] = userAgent;
    return _inner.send(request);
  }
}
```
