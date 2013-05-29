# Known Issues in SuperSocket 1.5

## Send data to client in the NewSessionConnected event handler

If you send data to client immediately in the NewSessionConnected event handler, a Null Reference exception will be thrown. You can change your code to send the welcome message to client in the override method of OnSessionStarted of your AppSession class instead.
