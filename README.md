# Demo instructions

Let's put some instructions here


## Node and VNC Setup
  See: [laptop and ssh configs](./sys-setup.md)
  

## EdgeRunner

## Left/Right
  [link to Left/Right demo code](./privacylayer-05012026/DEMO.md)

## PTZ Agent
  (put demo steps, what to do, what to say, etc, here)
  
  [link to agentic ptz code](./ptz-agent/)

  [PTZ Agent DEMO Steps](./ptz-agent/DEMO.md)

## BioClip
Run BioClip detection using ECR with this job config:

```
name: sage-vision-detect-bioclip-h015
plugins:
- name: sage-vision-detect
  pluginSpec:
    image: registry.sagecontinuum.org/plebbyd/sage-vision-detect:0.3.0
    args:
    - --stream
    - rtsp://camera:0Bscura%23@10.31.81.44:554/Preview_01_main
    - --name
    - lab_ptz
    - --backend
    - bioclip
    - --mode
    - caption
    volume: {}
nodeTags: []
nodes:
  H015: null
scienceRules:
- 'schedule(sage-vision-detect): cronjob("sage-vision-detect", "*/5 * * * *")'
successCriteria:
- WallClock(1d)
```

## Teton-PTZ-Agent-Sim

