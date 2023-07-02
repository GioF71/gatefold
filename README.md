## Browser App

The browser app uses a pub/sub model to listen for messages from the Raspberry Pi.
It can be served either from the Raspberry Pi itself or anywhere.

TODO: Make a service to serve the app

### Prerequisites

- Copy `.env.example` to `.env` and fill in the values. The MQTT values should match those in the Raspotify script.
- Run `npm install`

### Dev

- Run `npm run dev`

### Production

- Run `npm run build`
- Serve preview via `npm run preview`

## Raspotify Script

A node script that lives on your Raspberry Pi and runs whenever the Raspotify (librespot) event hook fires (eg when the Raspotify player changed songs).
This sends an event to the browser app to let it know that something happened.

### Part 1 - The event script

- Ensure Node is installed system-wide on your Raspberry Pi (nvm won't do)
- cd `raspotify/`
- Run `npm install`
- Configure the `topic` and `brokerUrl` in `gatefold.js`

Add this to your Raspotify config file (`/etc/raspotify/conf`):

```
LIBRESPOT_ONEVENT="node /home/pi/path/to/gatefold/raspotify/gatefold.js"
```

### Part 2 - The Raspotify service

At some point the Raspotify service was updated to only run in a high security state with very few permissions.
This prevents our script from being able to run (see [official discussion thread](https://github.com/dtcooper/raspotify/issues/500) on this issue). To fix this, we can overwrite the default
Raspotify service with our own one that has more freedom and runs under the `pi` user.

Caution: Run at your own risk.

- Backup the existing service: `sudo mv /lib/systemd/system/raspotify.service.backup`
- Copy our service in place of the original one: `sudo cp raspotify.service  /lib/systemd/system/`
- Reload systemd files and restart the Raspotify service: `sudo systemctl daemon-reload && sudo systemctl restart raspotify`

Raspotify should now be sending out MQTT messages whenever you pause/play (etc) a track.
You can confirm this by following the Raspotify logs in realtime:

`sudo journalctl —follow -u raspotify`

You should see it publish a MQTT message on each event.

### Dev

- Run `npm run dev` to publish a test event.

If the browser app is configured and running correctly, you should see it display a track.
If not, try opening the browser console and check if it's receiving any
messages/showing any errors.

## Useful links

- https://github.com/dtcooper/raspotify/issues/171#issuecomment-507423901
- https://github.com/librespot-org/librespot/blob/aa880f8888226a8e5fc6e1e54dfb7cf58176ac95/src/player_event_handler.rs
