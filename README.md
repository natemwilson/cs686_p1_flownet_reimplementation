# CS686 FlowNet Reimplementation

## usage

In order to view these examples, do the following:

clone this repository
```bash
$ git clone https://github.com/natemwilson/cs686_p1_flownet_reimplementation.git
```

navigate to the repository on the command line
```bash
$ cd cs686_p1_flownet_reimplementation
```

start your favorite http server, I use:
```bash
$ python3 -m http.server 8081
```

go to the link where the webserver is serving, in my case http://0.0.0.0:8081/

select any of the html files to see the visualization.

If you would like to look at one of the different datasets designed for minimal_force.html, replace the string `fabric2.json` with one of the following:
  * `bar.json`
  * `connected.json`
  * `fabric.json`
  * `miserables.json`
  * `tree.json`