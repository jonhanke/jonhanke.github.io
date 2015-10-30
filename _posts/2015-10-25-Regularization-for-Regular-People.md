---
layout: post
title: Regularization for regular people
---

<!-- extra_css:
  - d3_layout.css
 -->

The idea of “regularization" is often spoken about in the world 
of Machine Learning as a way of “tuning" models or “preventing overfitting",
but the precise meaning of this is usually not spoken about, or at least 
not in terms that any regular person would be happy to hear.  

In this blog post, we'll take a look at the meaning of regularization 
and work out in full detail all of the nuances of how various kinds 
(let's call them “flavors") of regularization allows us to answer the question:

\\[
\text{What is the “best" line going through a given point in the plane?}
\\]

{% comment %}
$$\text{Main Question: What is the ``best'' line going through a given point in the plane?}$$
$$
\sum_{i=1}^{\infty} \frac{1}{n^2} x\_0
$$
{% endcomment %}

<center>
<div border="black">
<svg id="Picture_many_lines">
</svg>
</div>
</center>


<!-- <script src="_posts/2015-10-25-Regularization.js"></script> -->


<script src="https://cdnjs.cloudflare.com/ajax/libs/d3/3.5.6/d3.min.js"></script>

<script> 

     function Draw_Picture1() {

        var svg = d3.select("svg");
        var svgHeight = 400;
        var svgWidth = 400;

        // Set margins and inner SVG box dimensions
        var margin = {top: 20, right: 10, bottom: 20, left: 10};
        var width = svgWidth - margin.left - margin.right,
            height = svgHeight - margin.top - margin.bottom;

        // Set the x-y plane region to show
        var xDomain = [-0.5, 2.5];
        var yDomain = [-0.5, 2.5];
        var myPoint = [1, 1];


        var xScale = d3.scale.linear()
                         .domain(xDomain)
                         .range([margin.left, width]);

        var xAxis = d3.svg.axis()
                      .scale(xScale)
                      // .outerTickSize(1)
                      .tickValues([1, 2])
                      // .tickFormat(",.0f")
                      .orient("bottom");


        var yScale = d3.scale.linear()
                         .domain(yDomain)
                         .range([height, margin.top]);

        var yAxis = d3.svg.axis()
                      .scale(yScale)
                      .tickValues([1, 2])
                      .orient("left");


        var myPointCoordinates = [xScale(myPoint[0]), yScale(myPoint[1])];


        // Taken from Mike Bostock's example http://bl.ocks.org/mbostock/3019563
        var defs = svg.append("defs");

            defs.append("marker")
                .attr("id", "triangle-start")
                .attr("viewBox", "0 0 10 10")
                .attr("refX", 10)
                .attr("refY", 5)
                .attr("markerWidth", -6)     // NOTE: This givens an error, but looks good! =)
                .attr("markerHeight", 6)
                .attr("orient", "auto")
              .append("path")
                .attr("d", "M 0 0 L 10 5 L 0 10 z");

            defs.append("marker")
                .attr("id", "triangle-end")
                .attr("viewBox", "0 0 10 10")
                .attr("refX", 10)
                .attr("refY", 5)
                .attr("markerWidth", 6)
                .attr("markerHeight", 6)
                .attr("orient", "auto")
              .append("path")
                .attr("d", "M 0 0 L 10 5 L 0 10 z");



        // Curly Brace taken from http://bl.ocks.org/alexhornbake/6005176
        //
        //returns path string d for <path d="This string">
        //a curly brace between x1,y1 and x2,y2, w pixels wide 
        //and q factor, .5 is normal, higher q = more expressive bracket 
        function makeCurlyBrace(x1,y1,x2,y2,w,q)
        {
            //Calculate unit vector
            var dx = x1-x2;
            var dy = y1-y2;
            var len = Math.sqrt(dx*dx + dy*dy);
            dx = dx / len;
            dy = dy / len;

            //Calculate Control Points of path,
            var qx1 = x1 + q*w*dy;
            var qy1 = y1 - q*w*dx;
            var qx2 = (x1 - .25*len*dx) + (1-q)*w*dy;
            var qy2 = (y1 - .25*len*dy) - (1-q)*w*dx;
            var tx1 = (x1 -  .5*len*dx) + w*dy;
            var ty1 = (y1 -  .5*len*dy) - w*dx;
            var qx3 = x2 + q*w*dy;
            var qy3 = y2 - q*w*dx;
            var qx4 = (x1 - .75*len*dx) + (1-q)*w*dy;
            var qy4 = (y1 - .75*len*dy) - (1-q)*w*dx;

        return ( "M " +  x1 + " " +  y1 +
                " Q " + qx1 + " " + qy1 + " " + qx2 + " " + qy2 + 
                " T " + tx1 + " " + ty1 +
                " M " +  x2 + " " +  y2 +
                " Q " + qx3 + " " + qy3 + " " + qx4 + " " + qy4 + 
                " T " + tx1 + " " + ty1 );
        }



        // Stylize the SVG with a border
        svg.attr("width", svgWidth)
            .attr("height", svgHeight)
            .style("border-style", "solid")
            .style("border-width", "0px")
            .style("border-color", "black");


        // Draw a line through the point P.
        var lineLength = 0.9;
        var lineLengthX = xScale(lineLength) - xScale(0);
        var lineAngles = [-30, 0, 30];
        // var lineAngles = [-45, -30, -15, 0, 15, 30, 45];
        svg.selectAll("line")
            .data(lineAngles)
            .enter()
                // .append("g")
                // .attr("stroke", "red")
                // .attr("fill", "red")
                .append("line")
                    .attr("x1", - lineLengthX)
                    .attr("y1", 0)
                    .attr("x2", lineLengthX)
                    .attr("y2", 0)
                    .attr("transform", function(d) { return "translate(" + myPointCoordinates[0] + ", " + myPointCoordinates[1] + ") rotate(" + -d + ")"; })
                    .attr("marker-start", "url(#triangle-start)")
                    .attr("marker-end", "url(#triangle-end)")
                    .attr("stroke", "black")
                    .attr("stroke-width", 2);
                    // .style("fill", "red");
                    // .selectAll("marker path")
                    //     .attr("stroke", "blue")
                    //     .style("fill", "blue");


        // // Draw the lines through the common point
        // line_data = [-150, -100, -50, -25, 0, 25, 50, 100, 150];
        // svg.selectAll("line")
        //     .data(line_data)
        //     .enter().append("line")
        //         .attr("x1", 100)
        //         .attr("y1", function(d) { return 200 + d; })
        //         .attr("x2", 300)
        //         .attr("y2", function(d) { return 200 - d; })
        //         .style("stroke", "blue")
        //         .style("stroke-width", 2);


        // Draw a curly brace
        var curly = {"x1": 1.9,  "y1": 0.5,  "x2": 1.9,  "y2": 1.5,  "w": 20,  "q":0.5};
        svg.append("path").attr("class","curlyBrace")
                .attr("d", makeCurlyBrace(xScale(curly.x1), yScale(curly.y1), 
                                            xScale(curly.x2), yScale(curly.y2), 
                                            curly.w, curly.q) )
                .style("stroke", "#000000")
                .style("stroke-width", "3px")
                .style("fill", "none");

        // Draw the ?? text
        svg.append("text")
            .text("??")
            .attr("x", xScale(2.1))
            .attr("y", yScale(1.0))
            .attr("dy", "0.3em")
            .attr("text-anchor", "left");

        // Draw the point on top
        svg.append("circle")
            .attr("cx", xScale(myPoint[0]))
            .attr("cy", yScale(myPoint[1]))
            .attr("r", 5)
            .attr("fill", "black");

        // Add the x-axis
        var xAxisGroup = svg.append("g")
            .attr("class", "axis")
            .attr("transform", "translate(0," + yScale(0) + ")")
            .call(xAxis);

        // // Attempt to add arrowheads
        // xAxisGroup.select("path")
        //         .attr("marker-start", "url(#triangle-start)")
        //         .attr("marker-end", "url(#triangle-end)");


        // Add the y-axis
        svg.append("g")
            .attr("class", "axis")
            .attr("transform", "translate(" + xScale(0) + ",0)")
            .call(yAxis);


        // Style the axes
        d3.selectAll(".axis line")
            .style("stroke", "black")
            .style("stroke-width", 2)
            .style("fill", "none")
            .style("shape-rendering", "crispEdges");

        d3.selectAll(".axis path")
            .style("stroke", "black")
            .style("stroke-width", 2)
            .style("fill", "none")
            .style("shape-rendering", "crispEdges");


        d3.selectAll(".axis text")
            .style("font-family", "sans-serif")
            .style("font-size", "11px");

    }

    Draw_Picture1();
</script>


<!-- <center>
![Many lines through a given point]({{ site.baseurl }}/images/jekyll-logo.png "Many lines through a given point")
</center>
 -->
## The Spirit of Regularization

We live in a modern and complicated world.  Things are happening around us everyday, 
and the more complicated the world gets the more choices we have to make.  Choices 
are usually advertised as a good thing, but at least from time-to-time they can be 
overwhelming.  Like when you walk into a supermarket thinking to yourself “I just 
want to get some bread and something to drink", but then find yourself confronted 
with dozens of kinds to choose from.  The problem changes from finding *something* 
to drink to finding the *best* thing to drink, and so you spend at least 5 minutes 
looking at all of the choices to evaluate which of these might be the best.  You might 
check to see that the drink isn't too expensive, doesn't have too much sugar, has 
enough real fruit juice to be called juice, or some not-so-bad tradeoff between these 
factors.

Once you get home and finally have time to relax with your drink, you might think 
to yourself that “Hey, I didn't really need all of that choice.  What I really wanted 
was something real, not too expensive, and not too sweet." If someone just handed that 
to me then I'd be happy.

That's kind of what regularization tries to do.  It hands you something “pretty good" 
by cutting down on the amount of freedom that you have.




## Flavors of Regularization

In the world of mathematical modelling, we usually have some data and we're trying 
to decide among possible ways of “best understanding" it from a certain perspective.  
The _perspective_ we use to understand the data is our given choice of mathematical model, 
and the way we have of _best understanding_ it is from this perspective what machine 
learning calls the “best fit" model.  It's the best choice of how to undersand the data 
with this model.

For purposes of this post, we'll focus on the perspective of understanding data given by 
points in the \\((x,y)\\)-plane by trying to draw a line through them that comes as close to predicting 
the \\(y\\)-values of all of them as closely as possible.  This perspective formally goes under the 
name of Linear Regression.

In general, once we have three points it's impossible to find a line that goes through all 
threee of them, and the “best" line is given by minimizing the “sum of the squares error" 
between the \\(y\\)-values of the line and the points for each of their \\(x\\)-values.  There are 
several well-known ways of finding this best line, but we'll settle here for a beautiful picture.


<center>
<div border="black">
<svg id="Picture_lines_through_three_points">
</svg>
</div>
</center>


<script> 

    function Draw_Picture2() {

        var svg = d3.select("#Picture_lines_through_three_points");
        var svgHeight = 400;
        var svgWidth = 400;

        // Set margins and inner SVG box dimensions
        var margin = {top: 20, right: 10, bottom: 20, left: 10};
        var width = svgWidth - margin.left - margin.right,
            height = svgHeight - margin.top - margin.bottom;

        // Set the x-y plane region to show
        var xDomain = [-0.5, 2.5];
        var yDomain = [-0.5, 2.5];
        var myPoint = [1, 1];


        var xScale = d3.scale.linear()
                         .domain(xDomain)
                         .range([margin.left, width]);

        var xAxis = d3.svg.axis()
                      .scale(xScale)
                      // .outerTickSize(1)
                      .tickValues([1, 2])
                      // .tickFormat(",.0f")
                      .orient("bottom");


        var yScale = d3.scale.linear()
                         .domain(yDomain)
                         .range([height, margin.top]);

        var yAxis = d3.svg.axis()
                      .scale(yScale)
                      .tickValues([1, 2])
                      .orient("left");


        var myPointCoordinates = [xScale(myPoint[0]), yScale(myPoint[1])];


        // Taken from Mike Bostock's example http://bl.ocks.org/mbostock/3019563
        var defs = svg.append("defs");

            defs.append("marker")
                .attr("id", "triangle-start")
                .attr("viewBox", "0 0 10 10")
                .attr("refX", 10)
                .attr("refY", 5)
                .attr("markerWidth", -6)     // NOTE: This givens an error, but looks good! =)
                .attr("markerHeight", 6)
                .attr("orient", "auto")
              .append("path")
                .attr("d", "M 0 0 L 10 5 L 0 10 z");

            defs.append("marker")
                .attr("id", "triangle-end")
                .attr("viewBox", "0 0 10 10")
                .attr("refX", 10)
                .attr("refY", 5)
                .attr("markerWidth", 6)
                .attr("markerHeight", 6)
                .attr("orient", "auto")
              .append("path")
                .attr("d", "M 0 0 L 10 5 L 0 10 z");





        // Stylize the SVG with a border
        svg.attr("width", svgWidth)
            .attr("height", svgHeight)
            .style("border-style", "solid")
            .style("border-width", "0px")
            .style("border-color", "black");


        // Draw a line through the point P.
        svg.append("line")
                    .attr("x1", xScale(0.5))
                    .attr("y1", yScale(0.5))
                    .attr("x2", xScale(1.5))
                    .attr("y2", yScale(1.5))
                    .attr("marker-start", "url(#triangle-start)")
                    .attr("marker-end", "url(#triangle-end)")
                    .attr("stroke", "black")
                    .attr("stroke-width", 2);
                    // .style("fill", "red");
                    // .selectAll("marker path")
                    //     .attr("stroke", "blue")
                    //     .style("fill", "blue");

        // Draw the ?? text
        svg.append("text")
            .text("??")
            .attr("x", xScale(2.1))
            .attr("y", yScale(1.0))
            .attr("dy", "0.3em")
            .attr("text-anchor", "left");

        // Draw the points on top
        var pointArray = [[1,1], [1.3,1], [0.5,0.3]];
        svg.selectAll("circle")
            .data(pointArray)
            .enter().append("circle")
                .attr("cx", function(d) { return xScale(d[0]);})
                .attr("cy", function(d) { return xScale(d[1]);})
                .attr("r", 5)
                .attr("fill", "black");

        // Add the x-axis
        var xAxisGroup = svg.append("g")
            .attr("class", "axis")
            .attr("transform", "translate(0," + yScale(0) + ")")
            .call(xAxis);

        // // Attempt to add arrowheads
        // xAxisGroup.select("path")
        //         .attr("marker-start", "url(#triangle-start)")
        //         .attr("marker-end", "url(#triangle-end)");


        // Add the y-axis
        svg.append("g")
            .attr("class", "axis")
            .attr("transform", "translate(" + xScale(0) + ",0)")
            .call(yAxis);


        // Style the axes
        d3.selectAll(".axis line")
            .style("stroke", "black")
            .style("stroke-width", 2)
            .style("fill", "none")
            .style("shape-rendering", "crispEdges");

        d3.selectAll(".axis path")
            .style("stroke", "black")
            .style("stroke-width", 2)
            .style("fill", "none")
            .style("shape-rendering", "crispEdges");


        d3.selectAll(".axis text")
            .style("font-family", "sans-serif")
            .style("font-size", "11px");

    }


    Draw_Picture2();

</script>


<!-- <center>
![Best line through three points]({{ site.baseurl }}/images/jekyll-logo.png "Best line through three points")
</center>
 -->


If you want to play with this, you can pick three points in the plane 
and the app will update to show the best line through them.


In addition to this “standard" minimization problem of finding the best line, there are 
two flavors of regluarization that people often impose to give preference to lines 
whose coefficients are small.  These are somewhat unintuitively called “\\(L^1\\)" and “\\(L^2\\)" 
regularization (or equally unintuitively “Lasso" and “Ridge" regression).  They involve 
imposing a certain kind of penalty for choosing lines whose coefficients are large.  Mathematically 
this involves adding a penalty that is either the “\\(L^1\\)"-norm or “\\(L^2\\)"-norm 
of the coefficients, and has the general cost function formulas

{% comment %}
$$ J_2(\beta_0, \beta_1, \dots, \beta_r) = \sum_{i=1}^n (\beta_0 + \beta_1 x_1^{(i)} + \cdots + \beta_r x_r^{(i)} - y^{(i)})^2$$
{% endcomment %}


\\[
J\_2(m,b) = \underbrace{\sum\_{i=1}^n ((mx^{(i)} + b) - y^{(i)})^2}\_{\text{Sum of squared errors} } 
+ \underbrace{\lambda(m^2 + b^2)}\_{\text{$L^2$ term} }
\\]


\\[
J\_1(m,b) = \underbrace{\sum\_{i=1}^n ((mx^{(i)} + b) - y^{(i)})^2}\_{\text{Sum of squared errors} } 
+ \underbrace{\lambda(|m| + |b|)}\_{\text{$L^1$ term} }
\\]

{% comment %}
where we're given $n$ data points named $P^{(i)}$ with coordinates $(x^{(i)}, y^{(i)})$.
{% endcomment %}

If this is not so enlightning, don't worry, most people would agree with you!  In the next section 
we'll explain what this means and what it does in the simplest possible example -- the 
underdetermined line through one point.  For now I'll just remind you of the often repeated 
general wisdom about these regularizations:  “\\(L^1\\)" is for feature selection and “\\(L^2\\)" is for small coefficients.



## The “best" line through one point

Imagine that you have one point in the plane.  One lonely isolated data point.  Let's say it's the point 
\\(P = (2,1)\\) for now just to be concrete (i.e. \\(x=2\\) and \\(y=1\\)).  If this is the only data point 
we have, then it's not very hard to find a line (of the form \\(y = mx + b\\) for some numbers 
\\(m\\) and \\(b\\) going through this point.  In fact there are lots of lines through the point \\(P -\\) infinitely 
many of them \\(-\\) and we can see this from playing with the following picture:



<center>
<div border="black">
<p>
  <label for="nAngle" 
     style="display: inline-block; width: 240px; text-align: right">
     angle = <span id="nAngle-value">…</span>
  </label>
  <input type="range" min="0" max="180" id="nAngle">
</p>
<svg id="Vary_the_line_through_one_point">
</svg>
</div>
</center>



<script> 




    function Draw_Picture3() {

        var svg = d3.select("#Vary_the_line_through_one_point");
        var svgHeight = 400;
        var svgWidth = 800;

        // Set margins and inner SVG box dimensions
        var margin = {top: 20, right: 10, bottom: 20, left: 10};
        var width = svgWidth - margin.left - margin.right,
            height = svgHeight - margin.top - margin.bottom;

        // Set the x-y plane region to show
        var xDomain = [-0.5, 4.5];
        var yDomain = [-0.5, 2.5];
        var myPoint = [2, 1];


        var xScale = d3.scale.linear()
                         .domain(xDomain)
                         .range([margin.left, width]);

        var xAxis = d3.svg.axis()
                      .scale(xScale)
                      // .outerTickSize(1)
                      .tickValues([1, 2, 3, 4])
                      // .tickFormat(",.0f")
                      .orient("bottom");


        var yScale = d3.scale.linear()
                         .domain(yDomain)
                         .range([height, margin.top]);

        var yAxis = d3.svg.axis()
                      .scale(yScale)
                      .tickValues([1, 2])
                      .orient("left");


        var myPointCoordinates = [xScale(myPoint[0]), yScale(myPoint[1])];




        // Stylize the SVG with a border
        svg.attr("width", svgWidth)
            .attr("height", svgHeight)
            .style("border-style", "solid")
            .style("border-width", "0px")
            .style("border-color", "black");


        // Draw a line through the point P.
        var myLine = svg.append("line")
                    .attr("x1", -190)
                    .attr("y1", 0)
                    .attr("x2", 190)
                    .attr("y2", 0)
                    .attr("marker-start", "url(#triangle-start)")
                    .attr("marker-end", "url(#triangle-end)")
                    .attr("stroke", "black")
                    .attr("stroke-width", 2)
                    .attr("transform", "translate(" + myPointCoordinates[0] + ", " + myPointCoordinates[1] + ")");
                    // .style("fill", "red");
                    // .selectAll("marker path")
                    //     .attr("stroke", "blue")
                    //     .style("fill", "blue");

        // Draw the points on top
        var pointArray = [[2,1]];
        svg.selectAll("circle")
            .data(pointArray)
            .enter().append("circle")
                .attr("cx", function(d) { return xScale(d[0]);})
                .attr("cy", function(d) { return yScale(d[1]);})
                .attr("r", 5)
                .attr("fill", "black");

        // Add the x-axis
        var xAxisGroup = svg.append("g")
            .attr("class", "x axis")
            .attr("transform", "translate(0," + yScale(0) + ")")
            .call(xAxis);

        // // Attempt to add arrowheads
        // xAxisGroup.select("path")
        //         .attr("marker-start", "url(#triangle-start)")
        //         .attr("marker-end", "url(#triangle-end)");


        // Add the y-axis
        svg.append("g")
            .attr("class", "y axis")
            .attr("transform", "translate(" + xScale(0) + ",0)")
            .call(yAxis);


        // Style the axes
        d3.selectAll(".axis line")
            .style("stroke", "black")
            .style("stroke-width", 2)
            .style("fill", "none")
            .style("shape-rendering", "crispEdges");

        d3.selectAll(".axis path")
            .style("stroke", "black")
            .style("stroke-width", 2)
            .style("fill", "none")
            .style("shape-rendering", "crispEdges");


        d3.selectAll(".axis text")
            .style("font-family", "sans-serif")
            .style("font-size", "11px");


        // =========== Slider code below here ================
        // Taken from http://bl.ocks.org/d3noob/10633421
        // ===================================================

        // when the input range changes update the angle 
        d3.select("#nAngle").on("input", function() {
          update(+this.value);
        });

        // Initial starting angle of the text 
        update(0);

        // update the element
        function update(nAngle) {

          // adjust the text on the range slider
          d3.select("#nAngle-value").text(nAngle);
          d3.select("#nAngle").property("value", nAngle);

          // rotate the text
          myLine
            .attr("transform", "translate(" + myPointCoordinates[0] + ", " + myPointCoordinates[1] + ")" 
                + " rotate("+nAngle+")");
        }



    }

    Draw_Picture3();

</script>


<!-- <center>
![Best line through three points]({{ site.baseurl }}/images/jekyll-logo.png "Best line through three points")
    [[ Graphic of (non-vertical) lines through the point P, where we can adjust the slope...]]
</center>
 -->


In the language of mathematical modelling, we would say that the problem of finding a line through the point \\(P\\) 
is *overfit* (since it passes exactly through all data points) and *underdetermined* (since there is not a unique 
line that does the job).  Because there are many lines that pass through \\(P\\), and they all seem to do the job, 
it's not possible to choose which among them is the “best" without imposing additional conditions to help say 
what we mean by “best".  Here is where the various flavors of regularization come in to help us to make that choice.


It's actually very helpful to be able to somehow see *all* lines in the plane at once, and then determine 
which of these lines in the plane passes through the given point \\(P\\).  One great way to do this is to think about 
lines in the plane as all being of the form \\(y=mx+b\\) for some numbers \\(m\\) and \\(b\\) (i.e. the slope and intercept of the line) 
and imagine a separate \\((m,b)\\)-plane.  This is the “parameter space" of lines in the plane, and here any point \\((m,b)\\) in the 
\\((m,b)\\)-plane corresponds to one line (i.e. \\(y=mx+b\\)) in the usual \\((x,y)\\)-plane.  It's kind of fun to be able to talk about all lines 
in the plane at once, but this “parameter space" language also allows us to write down explicitly which lines in the plane
pass through the given point \\(P\\).  It's the points in the \\((m,b)\\)-plane that describe lines where \\(y = mx + b\\) 
have the point \\(P = (x,y) = (2,1)\\), or equivalently where \\(1 = m \cdot 2 + b\\).

{% comment %}
\\[
\text{$y = mx + b$ have the point $P = (x,y) = (2,1)$, 
or equivalently where $1 = m \cdot 2 + b$.}
\\]
{% endcomment %}


This means that in the \\((m,b)\\) “parameter space" plane the lines passing through the point \\(P = (2,1)\\) are described by 
the points that satisfy the equation \\(2m + b = 1\\).  The fact that there are infinitely many points on the line \\(2m+b=1\\)
says exactly that there are infinitely many lines in the \\((x,y)\\)-plane passing through the point \\(P = (2,1)\\).



<center>
<div border="black">
<svg id="m_b_plane">
</svg>
</div>
</center>



<script> 




    function Draw_Picture4() {

        var svg = d3.select("#m_b_plane");
        var svgHeight = 400;
        var svgWidth = 400;

        // Set margins and inner SVG box dimensions
        var margin = {top: 20, right: 10, bottom: 20, left: 10};
        var width = svgWidth - margin.left - margin.right,
            height = svgHeight - margin.top - margin.bottom;

        // Set the x-y plane region to show
        var xDomain = [-0.5, 1.5];
        var yDomain = [-0.5, 1.5];
        var myPoint = [2, 1];


        var xScale = d3.scale.linear()
                         .domain(xDomain)
                         .range([margin.left, width]);

        var xAxis = d3.svg.axis()
                      .scale(xScale)
                      // .outerTickSize(1)
                      .tickValues([0.5, 1])
                      // .tickFormat(",.0f")
                      .orient("bottom");


        var yScale = d3.scale.linear()
                         .domain(yDomain)
                         .range([height, margin.top]);

        var yAxis = d3.svg.axis()
                      .scale(yScale)
                      .tickValues([0.5, 1, 1.5])
                      .orient("left");


        var myPointCoordinates = [xScale(myPoint[0]), yScale(myPoint[1])];




        // Stylize the SVG with a border
        svg.attr("width", svgWidth)
            .attr("height", svgHeight)
            .style("border-style", "solid")
            .style("border-width", "0px")
            .style("border-color", "black");


        // Draw a line through the point P.
        var myLine = svg.append("line")
                    .attr("x1", xScale(-0.25))
                    .attr("y1", yScale(1.5))
                    .attr("x2", xScale(0.75))
                    .attr("y2", yScale(-0.5))
                    .attr("marker-start", "url(#triangle-start)")
                    .attr("marker-end", "url(#triangle-end)")
                    .attr("stroke", "black")
                    .attr("stroke-width", 2);
                    // .style("fill", "red");
                    // .selectAll("marker path")
                    //     .attr("stroke", "blue")
                    //     .style("fill", "blue");

        // Add the x-axis
        var xAxisGroup = svg.append("g")
            .attr("class", "x axis")
            .attr("transform", "translate(0," + yScale(0) + ")")
            .call(xAxis);

        xAxisGroup.append("text")
            .text("m")
            .attr("x", xScale(1.4))
            .attr("y", 20)
            .attr("text-anchor", "center");


        // // Attempt to add arrowheads
        // xAxisGroup.select("path")
        //         .attr("marker-start", "url(#triangle-start)")
        //         .attr("marker-end", "url(#triangle-end)");


        // Add the y-axis
        var yAxisGroup = svg.append("g")
            .attr("class", "y axis")
            .attr("transform", "translate(" + xScale(0) + ",0)")
            .call(yAxis);

        yAxisGroup.append("text")
            .text("b")
            .attr("x", 10)
            .attr("y", yScale(1.4))
            .attr("text-anchor", "center");


        // Style the axes
        svg.selectAll(".axis line")
            .style("stroke", "red")
            .style("stroke-width", 2)
            .style("fill", "none")
            .style("shape-rendering", "crispEdges");

        svg.selectAll(".axis path")
            .style("stroke", "red")
            .style("stroke-width", 2)
            .style("fill", "none")
            .style("shape-rendering", "crispEdges");


        svg.selectAll(".axis text")
            .style("font-family", "sans-serif")
            .style("font-size", "11px");

    }

    Draw_Picture4();

</script>

<!-- 
<center>
![Best line through three points]({{ site.baseurl }}/images/jekyll-logo.png "Best line through three points")
     [[ Picture of 2m + b = 1 in the m-b plane, and show how various points 
        correspond to lines through (2,1) in the x-y plane. ]]
</center>
 -->



With this picture in hand, we're ready to see the effect of various flavors of regualrization on determining their 
own version of a “best" line.



## The \\(L^2\\)-best line

From the perspective of \\(L^2\\)-regularization, the best line is the one that minimizes the \\(L^2\\) cost function 

\\[
J\_2(m,b) = \underbrace{\sum\_{i=1}^n ((mx^{(i)} + b) - y^{(i)})^2}\_{\text{Sum of squared errors} } 
+ \underbrace{\lambda(m^2 + b^2)}\_{\text{$L^2$ term} }
\\]

<!-- \\[
BLAH = SSE + L^2-term
\\]
 -->

This formula might feel a little unwieldy, but in our case there is just one data point, and since any line we consider 
passes through the data poing the \\(y\\)-value of the line and the point agree (at the \\(x\\)-value of the point).  This means that the 
sum of squared errors is zero, so we're really minimizing the function.  Here no matter what the positive constant \\(\lambda\\) is 
the function will be minimized at the same point, so we're reduced to trying to find the line in the \\((x,y)\\)-plane through the 
point \\(P=(2,1)\\) where the function \\(J\_2(m,b) = m^2 + b^2\\) is the smallest.

Now here the cool part is that we know what the level sets of this function \\(J\_2\\) looks like in the \\((m,b)\\)-plane.  They are given 
by circles centered at the origin, and the value of \\(J\_2\\) grows as the radius of the circle grows.  Therefore to find the “\\(L^2\\)-best" 
line through \\(P=(2,1)\\) we need to find the smallest radius circle centered at the origin that touches the line \\(2m + b = 1\\) in the 
\\((m,b)\\)-plane. 


<center>
<div border="black">
<p>
  <label for="jTwo" 
     style="display: inline-block; width: 240px; text-align: right">
     \\(J\_2(m,b)\\) = <span id="j1-value">…</span>
  </label>
  <input type="range" step="0.01" min="0" max="1.2" id="jTwo">
</p>
<svg id="m_b_plane_with_circle">
</svg>
</div>
</center>



<script> 




    function Draw_Picture5() {

        var svg = d3.select("#m_b_plane_with_circle");
        var svgHeight = 600;
        var svgWidth = 600;

        // Set margins and inner SVG box dimensions
        var margin = {top: 20, right: 10, bottom: 20, left: 10};
        var width = svgWidth - margin.left - margin.right,
            height = svgHeight - margin.top - margin.bottom;

        // Set the x-y plane region to show
        var xDomain = [-0.5, 1.5];
        var yDomain = [-0.5, 1.5];
        var myPoint = [2, 1];


        var xScale = d3.scale.linear()
                         .domain(xDomain)
                         .range([margin.left, width]);

        var xAxis = d3.svg.axis()
                      .scale(xScale)
                      // .outerTickSize(1)
                      .tickValues([0.5, 1])
                      // .tickFormat(",.0f")
                      .orient("bottom");


        var yScale = d3.scale.linear()
                         .domain(yDomain)
                         .range([height, margin.top]);

        var yAxis = d3.svg.axis()
                      .scale(yScale)
                      .tickValues([0.5, 1, 1.5])
                      .orient("left");


        var myPointCoordinates = [xScale(myPoint[0]), yScale(myPoint[1])];




        // Stylize the SVG with a border
        svg.attr("width", svgWidth)
            .attr("height", svgHeight)
            .style("border-style", "solid")
            .style("border-width", "0px")
            .style("border-color", "black");


        // Draw a moduli line
        var myLine = svg.append("line")
                    .attr("x1", xScale(-0.25))
                    .attr("y1", yScale(1.5))
                    .attr("x2", xScale(0.75))
                    .attr("y2", yScale(-0.5))
                    .attr("marker-start", "url(#triangle-start)")
                    .attr("marker-end", "url(#triangle-end)")
                    .attr("stroke", "black")
                    .attr("stroke-width", 2);
                    // .style("fill", "red");
                    // .selectAll("marker path")
                    //     .attr("stroke", "blue")
                    //     .style("fill", "blue");


        // Draw a fitting circle
        var myCircle = svg.append("circle")
                        .attr("cx", xScale(0))
                        .attr("cy", yScale(0))
                        .attr("r", 50)
                        .attr("fill", "none")
                        .attr("stroke-dasharray", "5,5")
                        .attr("stroke", "blue")
                        .attr("stroke-width", 1);

    
        // Add the x-axis
        var xAxisGroup = svg.append("g")
            .attr("class", "x axis")
            .attr("transform", "translate(0," + yScale(0) + ")")
            .call(xAxis);

        xAxisGroup.append("text")
            .text("m")
            .attr("x", xScale(1.4))
            .attr("y", 20)
            .attr("text-anchor", "center");


        // // Attempt to add arrowheads
        // xAxisGroup.select("path")
        //         .attr("marker-start", "url(#triangle-start)")
        //         .attr("marker-end", "url(#triangle-end)");


        // Add the y-axis
        var yAxisGroup = svg.append("g")
            .attr("class", "y axis")
            .attr("transform", "translate(" + xScale(0) + ",0)")
            .call(yAxis);

        yAxisGroup.append("text")
            .text("b")
            .attr("x", 10)
            .attr("y", yScale(1.4))
            .attr("text-anchor", "center");


        // Style the axes
        svg.selectAll(".axis line")
            .style("stroke", "red")
            .style("stroke-width", 2)
            .style("fill", "none")
            .style("shape-rendering", "crispEdges");

        svg.selectAll(".axis path")
            .style("stroke", "red")
            .style("stroke-width", 2)
            .style("fill", "none")
            .style("shape-rendering", "crispEdges");


        svg.selectAll(".axis text")
            .style("font-family", "sans-serif")
            .style("font-size", "11px");


        // =========== Slider code below here ================
        // Taken from http://bl.ocks.org/d3noob/10633421
        // ===================================================

        // when the input range changes update the angle 
        d3.select("#jTwo").on("input", function() {
          updateJ1(+this.value);
        });

        // Initial starting angle of the text and empty array of intersection points
        updateJ1(1.0);
        var intersectionPointArray = [[1.0, 1.0], [1.1, 1.1]];


        // Draw intersection points
        var myPoints = svg.selectAll(".intersectionCircle")
                        .data(intersectionPointArray)
                        .enter().append("circle")
                            .attr("class", "intersectionCircle")
                            .attr("cx", function(d) { return xScale(d[0]);})
                            .attr("cy", function(d) { return yScale(d[1]);})
                            .attr("r", 5)
                            .attr("fill", "blue");



        // update the element
        function updateJ1(jTwo) {

          // create the new radius with two digit precision
          var rSqrt = Math.sqrt(jTwo);
          var rSqrt_trunc = (Math.round(rSqrt)*100)/100
          var jTwo_trunc = Math.round(jTwo*100)/100


          // adjust the text on the range slider
          d3.select("#j1-value").text(jTwo_trunc);
          d3.select("#jTwo").property("value", jTwo);

          // console.log(jOne);
          // console.log(Math.sqrt(jOne));

          // Change the circle radius
          myCircle
            .attr("r", xScale(Math.sqrt(jTwo)) - xScale(0));

          // Compute the intersection quadratic for m.
          //
          // m^2 + b^2 = r^2
          // 2*m + b = 1  -->  b = 1 - 2*m
          //   --> m^2 + (1-2*m)^2 = r^2
          //   --> m^2 + 1 - 4*m + 4*m^2 = r^2
          //   --> 5m^2 -4*m + 1-r^2 = 0 
          //
          var quad_a = 5;
          var quad_b = -4;
          var quad_c = 1-jTwo;
          var quad_disc = quad_b*quad_b - 4*quad_a*quad_c;

          // Change the circle coloring based on the number of roots
          console.log("quad_disc = ", quad_disc);
          if (quad_disc == 0.0) {
                myCircle.attr("stroke-dasharray", "none")
            } else {
                myCircle.attr("stroke-dasharray", "5,5")
            };



          // Compute the intersection point coordinates
          if (quad_disc < 0) {
            intersectionPointArray =[];
          } else {
            quad_disc_sqrt = Math.sqrt(quad_disc);
            var m1 = (-quad_b + quad_disc_sqrt) / (2 * quad_a);
            var b1 = 1 - 2*m1;
            var m2 = (-quad_b - quad_disc_sqrt) / (2 * quad_a);
            var b2 = 1 - 2*m2;
            var intersectionPointArray = [[m1, b1], [m2, b2]];

            console.log("intersectionPointArray = ", intersectionPointArray);

          }


          // Update intersection points
          var pts = svg.selectAll(".intersectionCircle").data(intersectionPointArray)
                  .attr("cx", function(d) { return xScale(d[0]);})
                  .attr("cy", function(d) { return yScale(d[1]);});

          pts.enter().append("circle")
                  .attr("class", "intersectionCircle")
                  .attr("cx", function(d) { return xScale(d[0]);})
                  .attr("cy", function(d) { return yScale(d[1]);})
                  .attr("r", 5)
                  .attr("fill", "blue");

          pts.exit()
                  .remove();


        }



    }

    Draw_Picture5();

</script>



<!-- <center>
![Circles growing to touch the parametrized line]({{ site.baseurl }}/images/jekyll-logo.png 
"Circles growing to touch the parametrized line")   
</center>
 -->


By drawing this, we can see that the circle tangent to the line \\(2m + b = 1\\) at this point, 
and so this radius of the circle is perpendicular 
to the line.  Therefore this intersection point lies on the line with inverse-reciprocal slope 
\\((\frac{-1}{\text{slope}} = \frac{-1}{-2} = \frac{1}{2})\\) and this line goes through the origin, 
so the intersection point lies on the line \\(b = \frac{m}{2}\\).  Solving both equations gives that the 
intersection point is \\((m,b) = (\frac{2}{5}, \frac{1}{5})\\), 
so the “\\(L^2\\)-best" line through the point \\(P = (2,1)\\) is the line \\(y = \frac{2}{5}x + \frac{1}{5}\\).

Notice that this is the line through \\(P\\) whose sum of squared of coefficients is smallest, or equivalently the line \\(y = mx+b\\) whose 
coefficient vector \\((m,b)\\) is the shortest among all possible lines through \\(P\\).



## The \\(L^1\\)-best line

From an \\(L^1\\)-regularization perspective, the best line is the one that minimizes the L^1 cost function

\\[
J\_1(m,b) = \underbrace{\sum\_{i=1}^n ((mx^{(i)} + b) - y^{(i)})^2}\_{\text{Sum of squared errors} } 
+ \underbrace{\lambda(|m| + |b|)}\_{\text{$L^1$ term} }
\\]

<!-- 
\\[ BLAH = SSE + L^1-term. \\]
 -->

For the same reasons as in the \\(L^2\\) case (i.e. the data is overfit, and \\(\lambda\\) can be taken to be 1) 
the SSE term is zero and we are left with minimizing the function \\(J\_1(m,b) = |m| + |b|\\).  The level sets 
of this function are diamonds (i.e. squares rotated by 45 degrees) centered at the origin, and again 
(as in the \\(L^2\\) case) the size of \\(J\_1(m,b)\\) grows with the size of the diamond.  Therefore the 
minimum value of \\(J\_1\\) 
for lines passing through the point \\(P = (2,1)\\) occurs where the smallest diamond touches the line \\(2m + b = 1\\) 
in the \\((m,b)\\)-plane.  



<center>
<div border="black">
<p>
  <label for="jOne" 
     style="display: inline-block; width: 240px; text-align: right">
     \\(J\_1(m,b)\\) = <span id="jOne-value">…</span>
  </label>
  <input type="range" step="0.01" min="0" max="1.2" id="jOne">
</p>
<svg id="m_b_plane_with_diamond">
</svg>
</div>
</center>



<script> 




    function Draw_Picture6() {

        var svg = d3.select("#m_b_plane_with_diamond");
        var svgHeight = 600;
        var svgWidth = 600;

        // Set margins and inner SVG box dimensions
        var margin = {top: 20, right: 10, bottom: 20, left: 10};
        var width = svgWidth - margin.left - margin.right,
            height = svgHeight - margin.top - margin.bottom;

        // Set the x-y plane region to show
        var xDomain = [-0.5, 1.5];
        var yDomain = [-0.5, 1.5];
        var myPoint = [2, 1];


        var xScale = d3.scale.linear()
                         .domain(xDomain)
                         .range([margin.left, width]);

        var xAxis = d3.svg.axis()
                      .scale(xScale)
                      // .outerTickSize(1)
                      .tickValues([0.5, 1])
                      // .tickFormat(",.0f")
                      .orient("bottom");


        var yScale = d3.scale.linear()
                         .domain(yDomain)
                         .range([height, margin.top]);

        var yAxis = d3.svg.axis()
                      .scale(yScale)
                      .tickValues([0.5, 1, 1.5])
                      .orient("left");


        var myPointCoordinates = [xScale(myPoint[0]), yScale(myPoint[1])];




        // Stylize the SVG with a border
        svg.attr("width", svgWidth)
            .attr("height", svgHeight)
            .style("border-style", "solid")
            .style("border-width", "0px")
            .style("border-color", "black");


        // Draw a moduli line
        var myLine = svg.append("line")
                    .attr("x1", xScale(-0.25))
                    .attr("y1", yScale(1.5))
                    .attr("x2", xScale(0.75))
                    .attr("y2", yScale(-0.5))
                    .attr("marker-start", "url(#triangle-start)")
                    .attr("marker-end", "url(#triangle-end)")
                    .attr("stroke", "black")
                    .attr("stroke-width", 2);
                    // .style("fill", "red");
                    // .selectAll("marker path")
                    //     .attr("stroke", "blue")
                    //     .style("fill", "blue");


        var L1DotColor = "purple";

        //This is the accessor function we talked about above
        var diamondFunction = d3.svg.line()
                          .x(function(d) { return xScale(d[0]); })
                          .y(function(d) { return yScale(d[1]); })
                         .interpolate("linear");

        // Make the closed diamond path
        function diamondPath(x) {
            return diamondFunction([[x,0], [0,x], [-x,0], [0, -x]]) + "Z";
        } 
    
        // Draw a fitting diamond
        var myDiamond = svg.append("path")
                            .attr("d", diamondPath(0.5))
                            .attr("fill", "none")
                            .attr("stroke-dasharray", "5,5")
                            .attr("stroke", L1DotColor)
                            .attr("stroke-width", 1);



        // Add the x-axis
        var xAxisGroup = svg.append("g")
            .attr("class", "x axis")
            .attr("transform", "translate(0," + yScale(0) + ")")
            .call(xAxis);

        xAxisGroup.append("text")
            .text("m")
            .attr("x", xScale(1.4))
            .attr("y", 20)
            .attr("text-anchor", "center");


        // // Attempt to add arrowheads
        // xAxisGroup.select("path")
        //         .attr("marker-start", "url(#triangle-start)")
        //         .attr("marker-end", "url(#triangle-end)");


        // Add the y-axis
        var yAxisGroup = svg.append("g")
            .attr("class", "y axis")
            .attr("transform", "translate(" + xScale(0) + ",0)")
            .call(yAxis);

        yAxisGroup.append("text")
            .text("b")
            .attr("x", 10)
            .attr("y", yScale(1.4))
            .attr("text-anchor", "center");


        // Style the axes
        svg.selectAll(".axis line")
            .style("stroke", "red")
            .style("stroke-width", 2)
            .style("fill", "none")
            .style("shape-rendering", "crispEdges");

        svg.selectAll(".axis path")
            .style("stroke", "red")
            .style("stroke-width", 2)
            .style("fill", "none")
            .style("shape-rendering", "crispEdges");


        svg.selectAll(".axis text")
            .style("font-family", "sans-serif")
            .style("font-size", "11px");


        // =========== Slider code below here ================
        // Taken from http://bl.ocks.org/d3noob/10633421
        // ===================================================

        // when the input range changes update the angle 
        d3.select("#jOne").on("input", function() {
          updateJ1(+this.value);
        });

        // Initial starting angle of the text and empty array of intersection points
        updateJ1(1.0);
        var intersectionPointArray = [[1.0, 1.0], [1.1, 1.1]];



        // Draw intersection points
        var myPoints = svg.selectAll(".intersectionCircle")
                        .data(intersectionPointArray)
                        .enter().append("circle")
                            .attr("class", "intersectionCircle")
                            .attr("cx", function(d) { return xScale(d[0]);})
                            .attr("cy", function(d) { return yScale(d[1]);})
                            .attr("r", 5)
                            .attr("fill", L1DotColor);



        // update the element
        function updateJ1(jOne) {

          // create the new radius with two digit precision
          var r = jOne;
          var r_trunc = Math.round(r*100)/100


          // adjust the text on the range slider
          d3.select("#jOne-value").text(r_trunc);
          d3.select("#jOne").property("value", r);

          // console.log(jOne);
          // console.log(Math.sqrt(jOne));

          // Change the circle radius
          myDiamond
            .attr("d", diamondPath(r));

          // Compute the intersection quadratic for m.
          //
          // m + b = r
          // 2*m + b = 1  -->  m = 1-r,  b = -1+2r
          //
          // b = m - r
          // 2*m + b = 1  -->  m = (1+r)/3,  b = (1-2r)/3
          //

          // Change the circle coloring based on the number of roots
          console.log("r = ", r);
          if (r == 0.5) {
                myDiamond.attr("stroke-dasharray", "none")
            } else {
                myDiamond.attr("stroke-dasharray", "5,5")
            };



          // Compute the intersection point coordinates
          if (r < 0.5) {
            intersectionPointArray =[];
          } else {
            var m1 = 1 - r;
            var b1 = -1 + 2*r;
            var m2 = (1 + r) / 3;
            var b2 = (1 - 2*r) / 3;
            var intersectionPointArray = [[m1, b1], [m2, b2]];

            console.log("intersectionPointArray = ", intersectionPointArray);

          }


          // Update intersection points
          var pts = svg.selectAll(".intersectionCircle").data(intersectionPointArray)
                  .attr("cx", function(d) { return xScale(d[0]);})
                  .attr("cy", function(d) { return yScale(d[1]);});

          pts.enter().append("circle")
                  .attr("class", "intersectionCircle")
                  .attr("cx", function(d) { return xScale(d[0]);})
                  .attr("cy", function(d) { return yScale(d[1]);})
                  .attr("r", 5)
                  .attr("fill", L1DotColor);

          pts.exit()
                  .remove();


        }



    }

    Draw_Picture6();

</script>



<!-- 
<center>
![Diamonds growing to touch the parametrized line]({{ site.baseurl }}/images/jekyll-logo.png 
"Diamonds growing to touch the parametrized line")   
</center>
 -->


From the picture we see that this happens when \\((m,b) = (\frac{1}{2}, 0)\\), so the \\(L^1\\)-best line through 
the point \\(P=(2,1)\\) is the line \\(y = \frac{1}{2}x\\).

We notice that this has selected the feature of the slope \\(m\\), and has selected against the \\(y\\)-intercept \\(b\\) 
(i.e. it preferred to have m non-zero and \\(b=0\\).  This gives an example of why it is said that \\(L^1\\)-regularization 
performs feature selection.  


## Conclusion

Finding the “best" line through a point is just one example of where regularization helps to choose a model, 
but this example should help to justify the general intuition about \\(L^2\\)- and \\(L^1\\)-regularization.  In 
these cases we see that \\(L^2\\)-regularization tries to make the coefficient vector small, 
while \\(L^1\\)-regularization helps to make the 
coefficients small but also has a strong preference for some coefficients to be zero.  



<center>
<div border="black">
<svg id="Regularized_lines_through_one_point">
</svg>
</div>
</center>



<script> 




    function Draw_Picture7() {

        var svg = d3.select("#Regularized_lines_through_one_point");
        var svgHeight = 400;
        var svgWidth = 800;

        // Set margins and inner SVG box dimensions
        var margin = {top: 20, right: 10, bottom: 20, left: 10};
        var width = svgWidth - margin.left - margin.right,
            height = svgHeight - margin.top - margin.bottom;

        // Set the x-y plane region to show
        var xDomain = [-0.5, 4.5];
        var yDomain = [-0.5, 2.5];
        var myPoint = [2, 1];


        var xScale = d3.scale.linear()
                         .domain(xDomain)
                         .range([margin.left, width]);

        var xAxis = d3.svg.axis()
                      .scale(xScale)
                      // .outerTickSize(1)
                      .tickValues([1, 2, 3, 4])
                      // .tickFormat(",.0f")
                      .orient("bottom");


        var yScale = d3.scale.linear()
                         .domain(yDomain)
                         .range([height, margin.top]);

        var yAxis = d3.svg.axis()
                      .scale(yScale)
                      .tickValues([1, 2])
                      .orient("left");


        var myPointCoordinates = [xScale(myPoint[0]), yScale(myPoint[1])];




        // Stylize the SVG with a border
        svg.attr("width", svgWidth)
            .attr("height", svgHeight)
            .style("border-style", "solid")
            .style("border-width", "0px")
            .style("border-color", "black");


        var startX = -0.4;
        var endX = 3.9;

        // Draw the best L2 line through the point P.
        function L2Line(x) { return (2*x + 1) / 5; };
        var myL2Line = svg.append("line")
                    .attr("x1", xScale(startX))
                    .attr("y1", yScale(L2Line(startX)))
                    .attr("x2", xScale(endX))
                    .attr("y2", yScale(L2Line(endX)))
                    .attr("marker-start", "url(#triangle-start)")
                    .attr("marker-end", "url(#triangle-end)")
                    .attr("stroke", "blue")
                    .attr("stroke-width", 2);
                    // .style("fill", "red");
                    // .selectAll("marker path")
                    //     .attr("stroke", "blue")
                    //     .style("fill", "blue");
        svg.append("text")
            .text("y=(2x+1)/5")
            // .text("\\(y=\frac{x}{2}\\)")
            .attr("x", xScale(endX + 0.1))
            .attr("y", yScale(L2Line(endX)))
            .attr("dy", "0.3em")
            .attr("text-anchor", "left")


        // Draw the best L1 line through the point P.
        function L1Line(x) { return x/2; };
        var myL1Line = svg.append("line")
                    .attr("x1", xScale(startX))
                    .attr("y1", yScale(L1Line(startX)))
                    .attr("x2", xScale(endX))
                    .attr("y2", yScale(L1Line(endX)))
                    .attr("marker-start", "url(#triangle-start)")
                    .attr("marker-end", "url(#triangle-end)")
                    .attr("stroke", "purple")
                    .attr("stroke-width", 2);
                    // .style("fill", "red");
                    // .selectAll("marker path")
                    //     .attr("stroke", "blue")
                    //     .style("fill", "blue");
        svg.append("text")
            .text("y=x/2")
            // .text("\\(y=\frac{x}{2}\\)")
            .attr("x", xScale(endX + 0.1))
            .attr("y", yScale(L1Line(endX)))
            .attr("dy", "0.3em")
            .attr("text-anchor", "left")



        // Draw the points on top
        var pointArray = [[2,1]];
        svg.selectAll("circle")
            .data(pointArray)
            .enter().append("circle")
                .attr("cx", function(d) { return xScale(d[0]);})
                .attr("cy", function(d) { return yScale(d[1]);})
                .attr("r", 5)
                .attr("fill", "black");

        svg.append("text")
            .text("P=(2,1)")
            // .text("\\(y=\frac{x}{2}\\)")
            .attr("x", xScale(2 - 0.1))
            .attr("y", yScale(1 - 0.2))
            .attr("dy", "0.3em")
            .attr("text-anchor", "center");



        // Add the x-axis
        var xAxisGroup = svg.append("g")
            .attr("class", "x axis")
            .attr("transform", "translate(0," + yScale(0) + ")")
            .call(xAxis);

        // // Attempt to add arrowheads
        // xAxisGroup.select("path")
        //         .attr("marker-start", "url(#triangle-start)")
        //         .attr("marker-end", "url(#triangle-end)");


        // Add the y-axis
        svg.append("g")
            .attr("class", "y axis")
            .attr("transform", "translate(" + xScale(0) + ",0)")
            .call(yAxis);


        // Style the axes
        d3.selectAll(".axis line")
            .style("stroke", "black")
            .style("stroke-width", 2)
            .style("fill", "none")
            .style("shape-rendering", "crispEdges");

        d3.selectAll(".axis path")
            .style("stroke", "black")
            .style("stroke-width", 2)
            .style("fill", "none")
            .style("shape-rendering", "crispEdges");


        d3.selectAll(".axis text")
            .style("font-family", "sans-serif")
            .style("font-size", "11px");


    }

    Draw_Picture7();

</script>





In general it's a not as clear to “see" what's going on, but one can imagine that it's something 
that “very similar to" what we see in this example.  Hopefully this kind of intuition is helpful 
to the reader when deciding what kinds of regularization(s) to use when approaching a model, 
and particularly a model that is overfit to some given data.  Thanks for reading!


## Exercises 

For the reader looking to try their hand at working out some examples on their own, I'd recommend the following exercises:

- Give the \\(L^2\\) and \\(L^1\\) regularized lines through the point \\((1,3)\\).

- Give the \\(L^2\\) and \\(L^1\\) regularized lines through the point \\((1,1)\\) -- noting that there may not be a unique choice!

- Give the \\(L^2\\) and \\(L^1\\) regularized lines through the point \\((0,0)\\).

- For which points \\(P\\) in the \\((x,y)\\)-plane does \\(L^1\\) regularization select for the slope \\(m\\)?  for the slope \\(b\\)?  for neither/both?
       
- For which points \\(P\\) is the best \\(L^1\\) regularized line through \\(P\\) not unique?  What conditions does \\(L^1\\)-regularization impose at these points?




