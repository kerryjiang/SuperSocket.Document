# Get the Connected Event and Closed Event of a Connection

> __Keywords__: Connection, Session, Connected Event, Closed Event

## Register session open/close handlers by the method ConfigureSessionHandler of the host builder

    builder.ConfigureSessionHandler((s) =>
        {
            // things to do when the session just connects
        },
        (s) =>
        {
            // things to do after the session closes
        });