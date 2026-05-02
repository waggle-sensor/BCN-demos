# Overview of Demo Setup:  Laptop and Sage Node

## VNC

Laptop on remote network,assuming X11 DISPLAY=:2

`ssh -L 10000:localhost:5902 beckman@waggle-dev-node-h015`

One on the Sage node:

```vncserver -kill :2
vncserver :2 -geometry 1920x1080
```

On the local laptop, fire up TigerVNC or the MacOS native Screen Sharing and go to:

`vnc://localhost:10000`

You will be asked for a password


