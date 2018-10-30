# Project 1 Report
This is a written report on the first project, a re-implementation of a modern data visualization paper,  assigned in CS686, Data Visualization Research, at the University of San Francisco.

## Paper

The paper that I chose to re-implement was: *"Animated Edge Textures in Node-Link Diagrams: a Design Space and Initial Evaluation"*. It seemed like a good opportunity for re-implementation because it was created using `d3.js` and all the code was available on github.

## Scope

The chosen paper can be split up into three main parts that could feasibly be re-implemented, including (1) a description of their underlying model, (2) a tutorial on using their API, which is built on their model, and (3) a set of interesting examples created with their API.

After a bit of exploration, it became clear that re-implementing the underlying model would be quite a difficult task given that the paper includes very little implementation details and that the included code is  long, confusing, and lacks documentation. 

I also considered building some of their example graphs using their API. This option was quickly ruled out though because their API is so simple to use, that this project would not have been sufficiently involved.

Eventually it was decided that the scope of this project would be to try and recreate some functionality from this paper based on a simpler model, the specific weekly goals being:
  1. Create some sort of looping animations using `d3.js`
  2. Apply looping animation logic to edges in a network.
  3. Apply network visualization logic from previous week to a geographic visualization.
  4. Add various visual embellishments to animated edges to convey other orthogonal parameters of the edges.
  
## Discussion
For the discussion, I will refer to the four main goals laid out above in the scope section, and for each I will explain the challenges and then detail the eventual approach taken.

### Looping Animations in `d3.js`
At first I tried and failed in many ways to use the `d3.transition()` function. I reasoned that if I could trigger a transition moving one item from point A to point B, then re-trigger it, I would have a good primitive to work with.

In one such attempt, I called the `d3.transition()` function multiple times in a simple while loop. 
```javascript
// select object to manipulate here
var circle = svg
    .append("circle")
    .attr('cx', 0)
    .attr('cy', 0);

// trigger transitions to random location in inf. loop
while (true) {
    var x = random_int_in_range(0, width);
    var y = random_int_in_range(0, height);
    circle
        .transition()
        .attr('cx', 0)
        .attr('cy', 0);
}
```
This did not work, and instead only showed the very first transition. Although I am ultimately not sure why, I reasoned that it has to do with the transition function  running asynchronously, in the background.

After some research into asynchrony in JavaScript and d3.js, I came upon a function `d3.selection.each()`, which allows the user to call a specific function given specified instructions. Specifically, adding the string `"end"` as the first parameter to the `each()` call, specifies a function to be called on each member of the selection at the end of the transition. Wrapping the code I wanted repeated in a function, then making a recursive call was enough to start a looping animation. The code looked like this:
```javascript
function animate(cx1, cy1, cx2, cy2) {
    
    function repeat() {
        var circle = svg
            .append("circle")
            .attr('cx', cx1)
            .attr('cy', cy1);
    
        circle
         .transition()
         .attr('cx', cx2)
         .attr('cy', cy2)
         .each("end", repeat);  
    }
    
    repeat();
}
```
Calling the above `animate()` function yielded this animation:

![figure_1](https://media.giphy.com/media/25PvN3WTfmVNT1i1JG/giphy.gif)

This seemed like great progress for a weeks work. I now had a primitive of a circle moving from point A to point B, looped, which I reasoned could be built up into a network of animated directed edges, given a dataset. Just to be sure that I could generate multiple moving circles, I tested the animate function in a loop like so:

```javascript
var i;  
for (i = 0; i < n; i++) {  
   var cx0, cy0, cx1, cy1;  
  
  cx0 = random_int_in_range(0, width);  
  cy0 = random_int_in_range(0, height);  
  cx1 = random_int_in_range(0, width);  
  cy1 = random_int_in_range(0, height);  
  
  animate(cx0, cy0, cx1, cy1);  
}
```
I observed the following, indicating that I was successful in animating multiple circles between two arbitrary points.

![figure_2](https://media.giphy.com/media/3rnwsd4PZjuBaKlwtX/giphy.gif)

### Animated Edges in Networks
In the second week, I was determined to apply this animate function to a network of nodes and edges, so I started my research with an investigation of `d3.forceSimulation()`. 

During my research, I learned a few key facts about the force simulation which gave me a new idea about how to simulated looping animations. They were that (1) calling `d3.forceSimulation()` actually creates a simulation object that maintains positions and velocities for each particle in the simulation and (2) that the animated simulation is modeled as a static svg image that changes state with each tick of the model, as specified by a tick function.

I then decided to change course from using `d3.selection.transition()` to drive animations, and instead use the force simulation. 

I created a standard force simulation network by creating nodes and links in the usual way. I then added one circle for each edge at the initial node of the directed edge. I then added an iterator variable global to the tick function,  I did that by writing a function which draws a circle somewhere along an edge, as specified by the value of the iterator. The tick code looked like the following:
```javascript
var i = 0;  
function tick() {
    if (i === 100) 
        i = 0;  
    i++;
    
    // NOT SHOWN HERE: code req'd for updating nodes and links

    // redraw moving circle at next position along edge
    moving_circles  
       .attr("cx", function(d) {  
           var dx = d.source.x - d.target.x;        
           return d.source.x - (dx * (i/n));
           }  
        )  
       .attr("cy", function(d) {  
           var dy = d.source.y - d.target.y;  
           return d.source.y - (dy * (i/n));
           }
       )  
}
```

The function worked nicely. A bit of cleaning and use of `d3.json()` yielded the following output which could take an arbitrary network and animate its edges.

![figure_3](https://media.giphy.com/media/WgTMGVITYg5ku9rh5u/giphy.gif)

This example nicely show's how visualization of the direction of an edge contributes new insight about a network. There is a clear directionality of network flow across the network that would not be obvious without the animations.

Some issues I ran into with this implementation were that with the default settings, the simulation stopped after ~300 ticks. I was able to fix this by changing the alpha decay parameter of the simulation to a very low number, which brings up the number of ticks to a massive number, large enough for our purposes such that the animation continues to animate for a long time.

### Animated Edges in GeoNetworks

After coming up with the above output, applying it to a map seemed like a simple matter of reading about GeoVisualization, and finding a way to initialize and fix the node positions.

My previous research on d3.forceSimulation showed me I could initialize the node positions by simply setting their `node.fx` and `node.fy` properties. I was surprised to find out that this also fixed the node positions for the duration of the simulation. I also deactivate node dragging controls, since dragging wouldn't make sense with a network overlaid on top of a map.

My research into GeoVisualization showed me that all I needed to add was a projection, a map, and latitude and longitude values to each node in the data, and I could then place the nodes on locations of my choosing. That all worked pretty easily.

One other challenge that I dealt with was trying to use two separate `json` data sources. I tried various designs, but ran into issues related to the fact that the data loading functions are asynchronous. I eventually settled on the following design, which worked well for my purposes.
```javascript
d3.json("geo.json", function(error, graph) {  
    d3.json("us-states.json", function(json) {
        // insert code using both data sources here
    }
}
```

After working through these challenges and combining my insights, I was able to create the following visualization:

![figure_4](https://media.giphy.com/media/7FgnZCoc6VGudVE01C/giphy.gif)




### Conveying Multiple Attributes With Animated Edges

For my final week of work, I added some attributes to the link data, with the goal of visualizing them as part of the edge. I also wanted to clean up the coloring of the visualization.

Displaying the parameters was a simple matter of setting the attribute of interest as a function of the data attribute. My proof of concept shows the result of varying color and size.

I also cleaned up the colors of the visualization by using a selection of colors from colorbrewer.org.

The result was the following:

![figure_5](https://media.giphy.com/media/2Yla1TaFRzuUGsoltx/giphy.gif)


## Future Work

One aspect of the original paper's implementation that my implementation never achieved was the ability to draw multiple moving objects along an edge, and to vary the speed and frequencies of those objects independently. Since my implementation was based on the global ticking of the force simulation, changing the speed of any edge animation would have effected the speed of every other edge. I considered creating one separate force simulation for each edge, but then reasoned that that approach might cause other issues.
As a future goal, I would definitely try to refactor my implementation so as to better allow for varying the speed, rhythm, and count of objects moving across each edge.