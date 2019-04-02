---
title: "Transfer Learning in Keras"
date: 2017-02-25 00:00:00
excerpt: "Repurposing pre-trained deep learning networks for new problems"
categories: [dl]
tags: [deep learning, visualization, d3, keras, tensorflow]
featured_image: '/images/prediction.jpg'
scroll_image: '/images/prediction.jpg'
comments: true
share: true
---
<script src="//code.jquery.com/jquery.js"></script>
<style>

	path {
		stroke-width: 4;
		fill: none;
	}

	.axis path,
	.axis line {
		fill: none;
		stroke: grey;
		stroke-width: 2;
		shape-rendering: crispEdges;
	}


</style>

### Using Transfer Learning and Bottlenecking to Capitalize on State of the Art DNNs

As a data scientist, I'm really interested in how deep learning networks can be deployed in industry. Usually the problems are too niche, there isn't enough training data, or there is a lack of computing power. Transfer learning is a way to make deep neural networks more accessible by overcoming some of these challenges.

Transfer learning is modification of pre-trained neural networks to perform on new datasets that they were not trained on. In this project specifically, the pre-trained networks include [InceptionV3](https://arxiv.org/abs/1512.00567), [VGG16](https://arxiv.org/abs/1409.1556), and [ResNet50](https://arxiv.org/abs/1512.03385). These are all convolutional neural networks that were trained on [ImageNet](http://www.image-net.org/) and extremely successful in the competition at one time or another. The new dataset is CIFAR-10.

If you are interested in a more general description of transfer learning, check out my article on [Medium](https://medium.com/@galen.ballew/transferlearning-b65772083b47#.6hsk4ruvn) about this project. I write more about what situations transfer learning is effective, how to implement it, and how to optimize it using bottlenecking.  

If you are interested in the actual implementation and workflow of transfer learning, definetely take a look at the [project repository.](https://github.com/galenballew/transfer-learning) There is a detailed Jupyter Notebook that walks through each step of the process and also contains everything found in this article and the one on [Medium](https://medium.com/@galen.ballew/transferlearning-b65772083b47#.6hsk4ruvn).  

As a base case, this project includes a simple CNN that was trained for just a few epochs to show how much quicker pre-trained networks converge. The base case network was written in Keras and is fashioned after the original LeNet architecture, with the addition of dropout as a regularization technique.

Below is a d3.js visualization of training epoch vs validation loss/accuracy.  

 Click on the labels beneath the graph to toggle the lines.

<div id='d3div'></div>

As the graph shows, all three pre-trained networks start off better than the base case. The rate at which model accuracy increases seems to be faily consistent through all training epochs, across all models.  

This is the most surprising result from the project. I think it may be in part due to the simple nature of the CIFAR-10 dataset. Each image is 28x28x3 for a total of 2352 pixels. The ImageNet dataset consists of 200x200x3 or 120,000 pixels for each image. The complexity of the data is an order of magnitude larger. It is most likely the case that the difference in network architecture becomes more apparent with higher dimensional data. Likewise, the rate of training between networks may have more variation if trained on data more complex than CIFAR-10.

However, in industry, these networks (especially ResNet50 in this case) required extremely little training time and were relatively easy to implement. Once there is a proof of concept, it is a lot easier to write an optimized network that suits your needs (and maybe mimics the network you transfer learned from) than it is to both write and train from scratch.

In a similar vein, DeepMind recently published a [paper](http://www.pnas.org/content/early/2017/03/13/1611835114) that shows the concept of transfering the learning of reinforcement agents from environment to environment. Although the implementation is very different, the idea of taking advantage of previous learning is fundamental and I suspect it will become a pillar of deep learning as we move forward. 





<script src="https://d3js.org/d3.v3.min.js"></script>
<script>

var force = d3.layout.force()
    .charge(-120)
    .linkDistance(30)
    .size([width, height]);

force
      .start();

var	margin = {top: 30, right: -100, bottom: 70, left: 350},
	// width = 1000 - margin.left - margin.right,
	// height = 600 - margin.top - margin.bottom;
	width=1000,
	height=600;

var	x = d3.scale.linear().range([0, width]);
var	y0 = d3.scale.linear().range([height, 0]);
var	y1 = d3.scale.linear().range([height, 0]);

var	xAxis = d3.svg.axis().scale(x)
	.orient("bottom").ticks(10);

var	yAxisLeft = d3.svg.axis().scale(y0)
	.orient("left").ticks(20);

var	yAxisRight = d3.svg.axis().scale(y1)
	.orient("right").ticks(20);

var	inception_val_line = d3.svg.line()
	.x(function(d) { return x(d.epoch); })
	.y(function(d) { return y0(d.inception_val); });
var	inception_loss_line = d3.svg.line()
	.x(function(d) { return x(d.epoch); })
	.y(function(d) { return y1(d.inception_loss); });

var	vgg_val_line = d3.svg.line()
	.x(function(d) { return x(d.epoch); })
	.y(function(d) { return y0(d.vgg_val); });
var	vgg_loss_line = d3.svg.line()
	.x(function(d) { return x(d.epoch); })
	.y(function(d) { return y1(d.vgg_loss); });

var	resnet_val_line = d3.svg.line()
	.x(function(d) { return x(d.epoch); })
	.y(function(d) { return y0(d.resnet_val); });
var	resnet_loss_line = d3.svg.line()
	.x(function(d) { return x(d.epoch); })
	.y(function(d) { return y1(d.resnet_loss); });

var	keras_val_line = d3.svg.line()
	.x(function(d) { return x(d.epoch); })
	.y(function(d) { return y0(d.keras_val); });
var	keras_loss_line = d3.svg.line()
	.x(function(d) { return x(d.epoch); })
	.y(function(d) { return y1(d.keras_loss); });

var	svg = d3.select("#d3div")
	.append("svg")
		.attr("width", width + margin.left + margin.right)
		.attr("height", height + margin.top + margin.bottom)
	.append("g")
		.attr("transform",
		      "translate(" + margin.left + "," + margin.top + ")");
var data = [
{"epoch":"01","inception_val":0.52, "inception_loss": 1.43,
              "vgg_val":0.56, "vgg_loss": 1.35,
              "resnet_val":0.79, "resnet_loss": 0.70,
              "keras_val":0.56, "keras_loss": 1.21},
{"epoch":"02","inception_val":0.61, "inception_loss": 1.14,
              "vgg_val":0.66, "vgg_loss": 1.03,
              "resnet_val":0.87, "resnet_loss": 0.48,
              "keras_val":0.63, "keras_loss": 1.06},
{"epoch":"03","inception_val":0.65, "inception_loss": 1.04,
              "vgg_val":0.70, "vgg_loss": 0.91,
              "resnet_val":0.91, "resnet_loss": 0.35,
              "keras_val":0.66, "keras_loss": 0.99},
{"epoch":"04","inception_val":0.67, "inception_loss": 0.99,
              "vgg_val":0.72, "vgg_loss": 0.81,
              "resnet_val":0.93, "resnet_loss": 0.29,
              "keras_val":0.67, "keras_loss": 0.93},
{"epoch":"05","inception_val":0.67, "inception_loss": 0.96,
              "vgg_val":0.73, "vgg_loss": 0.82,
              "resnet_val":0.96, "resnet_loss": 0.23,
              "keras_val":0.68, "keras_loss": 0.91},
{"epoch":"06","inception_val":0.68, "inception_loss": 0.95,
              "vgg_val":0.75, "vgg_loss": 0.75,
              "resnet_val":0.97, "resnet_loss": 0.20,
              "keras_val":0.69, "keras_loss": 0.91},
{"epoch":"07","inception_val":0.68, "inception_loss": 0.94,
              "vgg_val":0.74, "vgg_loss": 0.81,
              "resnet_val":0.99, "resnet_loss": 0.14,
              "keras_val":0.71, "keras_loss": 0.86},
{"epoch":"08","inception_val":0.69, "inception_loss": 0.94,
              "vgg_val":0.76, "vgg_loss": 0.72,
              "resnet_val":0.99, "resnet_loss": 0.12,
              "keras_val":0.71, "keras_loss": 0.85},
{"epoch":"09","inception_val":0.69, "inception_loss": 0.93,
              "vgg_val":0.77, "vgg_loss": 0.70,
              "resnet_val":0.99, "resnet_loss": 0.11,
              "keras_val":0.71, "keras_loss": 0.85},
{"epoch":"10","inception_val":0.69, "inception_loss": 0.93,
              "vgg_val":0.77, "vgg_loss": 0.73,
              "resnet_val":1.0, "resnet_loss": 0.09,
              "keras_val":0.71, "keras_loss": 0.87},

];

// Get the data
data.forEach(function(d) {
	d.epoch = +d.epoch;
	d.inception_val = +d.inception_val;
	d.inception_loss = +d.inception_loss;
});

// Scale the range of the data
x.domain(d3.extent(data, function(d) { return d.epoch; }));
y0.domain([0, 1.0]);
y1.domain([0, 1.5]);

svg.append("path")
	.attr("class", "line")
	.style("stroke", "red")
	.attr("id", "InceptionV3_acc")
	.attr("d", inception_val_line(data));
svg.append("path")
	.attr("class", "line")
	.style("stroke", "red")
	.attr("id", "InceptionV3_loss")
	.attr("d", inception_loss_line(data));

svg.append("path")
	.attr("class", "line")
	.style("stroke", "steelblue")
	.attr("id", "VGG_acc")
	.attr("d", vgg_val_line(data));
svg.append("path")
	.attr("class", "line")
	.style("stroke", "steelblue")
	.attr("id", "VGG_loss")
	.attr("d", vgg_loss_line(data));

svg.append("path")
	.attr("class", "line")
	.style("stroke", "green")
	.attr("id", "ResNet_acc")
	.attr("d", resnet_val_line(data));
svg.append("path")
	.attr("class", "line")
	.style("stroke", "green")
	.attr("id", "ResNet_loss")
	.attr("d", resnet_loss_line(data));

svg.append("path")
	.attr("class", "line")
	.style("stroke", "orange")
	.attr("id", "Keras_acc")
	.attr("d", keras_val_line(data));
svg.append("path")
	.attr("class", "line")
	.style("stroke", "orange")
	.attr("id", "Keras_loss")
	.attr("d", keras_loss_line(data));


// x axis
svg.append("g")
	.attr("class", "x axis")
	.attr("transform", "translate(0," + height + ")")
	.call(xAxis);

// edit the Y Axis Left
svg.append("g")
	.attr("class", "y axis")
	.style("fill", "black")
	.attr("id", "blueAxis")
	.call(yAxisLeft);

svg.append("g")
	.attr("class", "y axis")
	.attr("transform", "translate(" + width + " ,0)")
	.style("fill", "black")
	.attr("id", "redAxis")
	.call(yAxisRight);

// inception lines
svg.append("text")
	.attr("x", 0)
	.attr("y", height + margin.top + 10)
	.attr("class", "legend")
	.style("fill", "red")
	.on("click", function(){
		// Determine if current line is visible
		var active   = InceptionV3_acc.active ? false : true,
		  newOpacity = active ? 0 : 1;
		// Hide or show the elements
		d3.select("#InceptionV3_acc").style("opacity", newOpacity);
		// Update whether or not the elements are active
		InceptionV3_acc.active = active;
	})
	.text("InceptionV3 Validation Accuracy");
svg.append("text")
	.attr("x", 0)
	.attr("y", height + margin.top + 30)
	.attr("class", "legend")
	.style("fill", "red")
	.on("click", function(){
		// Determine if current line is visible
		var active   = InceptionV3_loss.active ? false : true ,
		  newOpacity = active ? 0 : 1;
		// Hide or show the elements
		d3.select("#InceptionV3_loss").style("opacity", newOpacity);
		// Update whether or not the elements are active
		InceptionV3_loss.active = active;
	})
	.text("InceptionV3 Validation Loss");

// vgg lines
svg.append("text")
	.attr("x", 260)
	.attr("y", height + margin.top + 10)
	.attr("class", "legend")
	.style("fill", "steelblue")
	.on("click", function(){
		// Determine if current line is visible
		var active   = VGG_acc.active ? false : true,
		  newOpacity = active ? 0 : 1;
		// Hide or show the elements
		d3.select("#VGG_acc").style("opacity", newOpacity);
		// Update whether or not the elements are active
		VGG_acc.active = active;
	})
	.text("VGG Validation Accuracy");
svg.append("text")
	.attr("x", 260)
	.attr("y", height + margin.top + 30)
	.attr("class", "legend")
	.style("fill", "steelblue")
	.on("click", function(){
		// Determine if current line is visible
		var active   = VGG_loss.active ? false : true ,
		  newOpacity = active ? 0 : 1;
		// Hide or show the elements
		d3.select("#VGG_loss").style("opacity", newOpacity);
		// Update whether or not the elements are active
		VGG_loss.active = active;
	})
	.text("VGG Validation Loss");

// resnet text
svg.append("text")
	.attr("x", 470)
	.attr("y", height + margin.top + 10)
	.attr("class", "legend")
	.style("fill", "green")
	.on("click", function(){
		// Determine if current line is visible
		var active   = ResNet_acc.active ? false : true,
		  newOpacity = active ? 0 : 1;
		// Hide or show the elements
		d3.select("#ResNet_acc").style("opacity", newOpacity);
		// Update whether or not the elements are active
		ResNet_acc.active = active;
	})
	.text("ResNet Validation Accuracy");
svg.append("text")
	.attr("x", 470)
	.attr("y", height + margin.top + 30)
	.attr("class", "legend")
	.style("fill", "green")
	.on("click", function(){
		// Determine if current line is visible
		var active   = ResNet_loss.active ? false : true ,
		  newOpacity = active ? 0 : 1;
		// Hide or show the elements
		d3.select("#ResNet_loss").style("opacity", newOpacity);
		// Update whether or not the elements are active
		ResNet_loss.active = active;
	})
	.text("ResNet Validation Loss");

	// keras text
	svg.append("text")
		.attr("x", 700)
		.attr("y", height + margin.top + 10)
		.attr("class", "legend")
		.style("fill", "orange")
		.on("click", function(){
			// Determine if current line is visible
			var active   = Keras_acc.active ? false : true,
			  newOpacity = active ? 0 : 1;
			// Hide or show the elements
			d3.select("#Keras_acc").style("opacity", newOpacity);
			// Update whether or not the elements are active
			Keras_acc.active = active;
		})
		.text("Keras Validation Accuracy");
	svg.append("text")
		.attr("x", 700)
		.attr("y", height + margin.top + 30)
		.attr("class", "legend")
		.style("fill", "orange")
		.on("click", function(){
			// Determine if current line is visible
			var active   = Keras_loss.active ? false : true ,
			  newOpacity = active ? 0 : 1;
			// Hide or show the elements
			d3.select("#Keras_loss").style("opacity", newOpacity);
			// Update whether or not the elements are active
			Keras_loss.active = active;
		})
		.text("Keras Validation Loss");
</script>
