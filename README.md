[![docs](https://readthedocs.org/projects/websocket-client/badge/?style=flat)](https://websocket-client.readthedocs.io/)
[![Build Status](https://travis-ci.com/websocket-client/websocket-client.svg?branch=master)](https://travis-ci.com/websocket-client/websocket-client)
[![codecov](https://codecov.io/gh/websocket-client/websocket-client/branch/master/graph/badge.svg?token=pcXhUQwiL3)](https://codecov.io/gh/websocket-client/websocket-client)
[![Downloads](https://pepy.tech/badge/websocket-client)](https://pepy.tech/project/websocket-client)
[![PyPI version](https://badge.fury.io/py/websocket_client.svg)](https://badge.fury.io/py/websocket_client)

# websocket-client

The websocket-client module is a WebSocket client for Python. It provides access
to low level APIs for WebSockets. All APIs are for synchronous functions.

websocket-client supports only [hybi-13](https://tools.ietf.org/html/draft-ietf-hybi-thewebsocketprotocol-13).

## License

- LGPL version 2.1

## Documentation

This project's documentation can be found at
[https://websocket-client.readthedocs.io/](https://websocket-client.readthedocs.io/)

## Contributing

Please see the contribution guidelines at
[https://websocket-client.readthedocs.io/en/latest/contributing.html](https://websocket-client.readthedocs.io/en/latest/contributing.html)

## Installation

First, install the following dependencies:
- six
- backports.ssl\_match\_hostname for Python 2.x

You can install the dependencies with the command `pip install six` and
`pip install backports.ssl_match_hostname`

You can use either `python setup.py install` or `pip install websocket-client`
to install. This module is tested on Python 2.7 and Python 3.4+. Python 3
support was first introduced in version 0.14.0, but is a work in progress.

## Usage Tips

Check out the documentation's FAQ for additional guidelines:
[https://websocket-client.readthedocs.io/en/latest/faq.html](https://websocket-client.readthedocs.io/en/latest/faq.html)

### Performance

The `send` and `validate_utf8` methods are very slow in pure Python. You can
disable UTF8 validation in this library (and receive a performance enhancement)
with the `skip_utf8_validation` parameter. If you want to get better
performance, please install both numpy and wsaccel, and import them into your
project files - these other libraries will automatically be used when available.
Note that wsaccel can sometimes cause other issues.

### Long-lived Connection

Most real-world WebSockets situations involve longer-lived connections.
The WebSocketApp `run_forever` loop automatically tries to reconnect when a
connection is lost, and provides a variety of event-based connection controls.
The project documentation has
[additional examples](https://websocket-client.readthedocs.io/en/latest/examples.html)

```python
import websocket
try:
    import thread
except ImportError:
    import _thread as thread
import time

def on_message(ws, message):
    print(message)

def on_error(ws, error):
    print(error)

def on_close(ws):
    print("### closed ###")

def on_open(ws):
    def run(*args):
        for i in range(3):
            time.sleep(1)
            ws.send("Hello %d" % i)
        time.sleep(1)
        ws.close()
        print("thread terminating...")
    thread.start_new_thread(run, ())

if __name__ == "__main__":
    websocket.enableTrace(True)
    ws = websocket.WebSocketApp("ws://echo.websocket.org/",
                              on_open = on_open,
                              on_message = on_message,
                              on_error = on_error,
                              on_close = on_close)

    ws.run_forever()
```

### Short-lived Connection

This is if you want to communicate a short message and disconnect
immediately when done. For example, if you want to confirm that a WebSocket
server is running and responds properly to a specific request.
The project documentation has
[additional examples](https://websocket-client.readthedocs.io/en/latest/examples.html)

```python
from websocket import create_connection
ws = create_connection("ws://echo.websocket.org/")
print("Sending 'Hello, World'...")
ws.send("Hello, World")
print("Sent")
print("Receiving...")
result =  ws.recv()
print("Received '%s'" % result)
ws.close()
```

If you want to customize socket options, set sockopt, as seen below:

```python
from websocket import create_connection
ws = create_connection("ws://echo.websocket.org/",
                        sockopt=((socket.IPPROTO_TCP, socket.TCP_NODELAY),))
```

### Acknowledgements

Thanks to @battlemidget and @ralphbean for helping migrate this project to
Python 3.