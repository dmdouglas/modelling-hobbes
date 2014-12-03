# Browser Code

This is the browser file where we pull together different parts of the simulation and wrap it up in the browser.  We'll start out by importing our libraries and defining any global variables we might need.


		d3 = require './assets/d3.min.js'
		simulation = require './simulation.coffee.md'
		ui = require './assets/jquery-ui-1.11.2.custom/jquery-ui.js'
		running = true
		height = window.innerHeight #|| 600
		#width = window.innerWidth || 600
		width = window.innerWidth - (window.innerWidth / 4)
		space = null

		game = window.location.search.match /\?game=(.+)&?/

Now we generate the user interfaces for the simulation controls. This includes start and stop buttons for the simulation.


		jQuery ->
			$ = jQuery
			$( "#stop" ).button({
				text: false;
				icons: {
					primary: "ui-icon-stop"
					}
			})
				.on "click", () ->
					running = false
					#alert 'Simulation stopped'
					#preventDefault()
					$.defaultPrevented()

			$( "#start" ).button({
				icons: {
					primary: "ui-icon-play"
					}
				text: false;
			})
				.on "click", () ->
					running = true
					$.defaultPrevented()

			$( "#reset" ).button({
				text: false;
				icons: {
					primary: "ui-icon-eject"
					}
			})
				.on "click", () ->
					running = false
					populate()
					running = true
					$.defaultPrevented()

					
Next, we'll grab create our svg canvas that will contain our agents and add it to the DOM.

		canvas = d3.select("#space")
					.append("svg:svg")
					.attr("height", height)
					.attr("width", width)


Now we create the graphics for the simulation legend. We take the list of strategies and draw a list of circles with the name of the strategy they represent next to them.

		info = d3.select("#controls")
			      .append("svg:svg")
				  .attr("height", height)

		legend = simulation.strategies
		info.selectAll "circle"
			 .data(legend)
			 .enter()
			 .append("circle")
			 .attr "cx", (d, i) -> 50
			 .attr "cy", (d, i) -> (i * 50) + 25
			 .style "fill", (d) -> d.color 
			 .style "opacity", 0.5
			 .attr "r", 8
			 .attr "title", (d) -> d.name

		info.selectAll("text")
			 .data(legend)
			 .enter()
			 .append("text")
			 .text (d) -> d.name
			 .attr "x", (d, i) -> 75
			 .attr "y", (d, i) -> (i * 50) + 30


Now we need to create svg circles to represent our agents and bind them to the actual agents from the simulation.  The `d` in the functions here is the D3js accessor to the agent data.

	
		populate = () ->
			space = simulation.agents(height, width)
			canvas.selectAll "circle"
				.data space
				.enter().append "circle"
				.style "fill", (d) -> d.strategy.color 
				.style "opacity", 0.5
				.attr "r", 8
				.attr "cx", (d) -> d.x
				.attr "cy", (d) -> d.y
		populate()


We then write a loop where each svg circle triggers the move function for its bound agent.  


		move = () ->
			circles = canvas.selectAll "circle"
			circles.each (d) ->
				d = simulation.interact d
			.each (d) ->
				d = simulation.update d
			.transition()
			.duration 500
			.attr "cx", (d) -> d.x
			.attr "cy", (d) -> d.y
			.attr "r", (d) -> 
				if d.strategy.i is 3 then 10 else Math.max 3, (d.score / 10)
			.attr "title", (d) -> "#{d.strategy.name} - #{d.score}"
			.style "fill", (d) -> d.strategy.color
		

		run = () ->
			move() unless running is false


Finally, we run the loop continuous with a half second pause.


		setInterval run, 500
