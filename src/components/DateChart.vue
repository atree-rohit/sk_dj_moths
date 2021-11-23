<style>
	.tick text{
		font-size: 2em !important;
	}
</style>

<template>
	<div id="date-chart-continer"></div>
</template>

<script>
	import * as d3 from "d3"
	import moment from 'moment'
	export default {
		name:"date-chart",
		props: [ "observations" ],
		data() {
			return{
				height: "100" ,
				width: "100",
				xScale: {},
				xScale2: {},
				yScale: {},
				yScale2: {},
				bars1: {},
				xAxis: {},
				xAxisGroup: {},
				watch_init_flag: false,
			}
		},
		mounted(){
			let chart = this.BarChart(this.months_data, {
				x: d => d.month,
				y: d => d.frequency,
				xDomain: this.months_data.map(d => d.month),
				yFormat: "%",
				yLabel: "",
				color: "steelblue"
			})
			console.log(chart)
			// let svg = d3.select("#date-chart-continer").append(chart)
		},
		computed:{
			months_data() {
				let op = []
				for(let i = 0; i<12 ; i++){
					op.push({
						month: moment().month(i).format("MMM"),
						frequency: 0
					})
				}
				this.observations.forEach(o => {
					op.forEach((m,i) => {
						if(m.month == moment(o.observed_on).format("MMM")){
							op[i].frequency++
						}
					})
				})
				return op
			}

		},
		watch: {
		},
		methods:{
			BarChart(data, {
			  x = (d, i) => i, // given d in data, returns the (ordinal) x-value
			  y = d => d, // given d in data, returns the (quantitative) y-value
			  title, // given d in data, returns the title text
			  marginTop = 20, // the top margin, in pixels
			  marginRight = 0, // the right margin, in pixels
			  marginBottom = 30, // the bottom margin, in pixels
			  marginLeft = 40, // the left margin, in pixels
			  width = 640, // the outer width of the chart, in pixels
			  height = 400, // the outer height of the chart, in pixels
			  xDomain, // an array of (ordinal) x-values
			  xRange = [marginLeft, width - marginRight], // [left, right]
			  yType = d3.scaleLinear, // y-scale type
			  yDomain, // [ymin, ymax]
			  yRange = [height - marginBottom, marginTop], // [bottom, top]
			  xPadding = 0.1, // amount of x-range to reserve to separate bars
			  yFormat, // a format specifier string for the y-axis
			  yLabel, // a label for the y-axis
			  color = "currentColor" // bar fill color
			} = {}) {
			  // Compute values.
			  const X = d3.map(data, x);
			  const Y = d3.map(data, y);

			  // Compute default domains, and unique the x-domain.
			  if (xDomain === undefined) xDomain = X;
			  if (yDomain === undefined) yDomain = [0, d3.max(Y)];
			  xDomain = new d3.InternSet(xDomain);

			  // Omit any data not present in the x-domain.
			  const I = d3.range(X.length).filter(i => xDomain.has(X[i]));

			  // Construct scales, axes, and formats.
			  const xScale = d3.scaleBand(xDomain, xRange).padding(xPadding);
			  const yScale = yType(yDomain, yRange);
			  const xAxis = d3.axisBottom(xScale).tickSizeOuter(0);
			  const yAxis = d3.axisLeft(yScale).tickFormat(d3.format('.0f')).ticks(d3.max(Y));

			  // Compute titles.
			  if (title === undefined) {
			    const formatValue = yScale.tickFormat(100, yFormat);
			    title = i => `${X[i]}\n${formatValue(Y[i])}`;
			  } else {
			    const O = d3.map(data, d => d);
			    const T = title;
			    title = i => T(O[i], i, data);
			  }

			  // const svg = d3.create("svg")
			  const svg = d3.select("#date-chart-continer").append("svg")
			      .attr("width", width)
			      .attr("height", height)
			      .attr("viewBox", [0, 0, width, height])
			      .attr("style", "max-width: 100%; height: auto; height: intrinsic;");

			  svg.append("g")
			      .attr("transform", `translate(${marginLeft},0)`)
			      .call(yAxis)
			      .call(g => g.select(".domain").remove())
			      .call(g => g.selectAll(".tick line").clone()
			          .attr("x2", width - marginLeft - marginRight)
			          .attr("stroke-opacity", 0.1))
			      .call(g => g.append("text")
			          .attr("x", -marginLeft)
			          .attr("y", 10)
			          .attr("fill", "currentColor")
			          .attr("text-anchor", "start")
			          .text(yLabel));

			  const bar = svg.append("g")
			      .attr("fill", color)
			    .selectAll("rect")
			    .data(I)
			    .join("rect")
			      .attr("x", i => xScale(X[i]))
			      .attr("y", i => yScale(Y[i]))
			      .attr("height", i => yScale(0) - yScale(Y[i]))
			      .attr("width", xScale.bandwidth());

			  if (title) bar.append("title")
			      .text(title);

			  svg.append("g")
			      .attr("transform", `translate(0,${height - marginBottom})`)
			      .call(xAxis);

			  // return svg.node();
			},
			init () {
				this.xScale = {}
				this.xScale2 = {}
				this.yScale = {}
				this.yScale2 = {}
				this.bars1 = {}
				this.xAxis = {}
				this.xAxisGroup = {}
				
				this.renderChart()
			},
			renderChart(){
				let margin = {top: 20, right: 20, bottom: 90, left: 50}
				let margin2 = {top: this.height - 70, right: 20, bottom: 30, left: 50}
				this.width = this.width - margin.left - margin.right
				this.height = this.height - margin.top - margin.bottom
				let height2 = 50

				let scaleFactor = 20

				if (!d3.select("#date-chart-continer svg").empty()) {
					d3.selectAll("#date-chart-continer svg").remove()
				}

				let svg = d3.select("#date-chart-continer").append("svg")
					// .attr("viewBox", [0, 0, this.width+margin.left+margin.right, this.height+margin.top+margin.bottom])
						.attr("width",this.width+margin.left+margin.right)
						.attr("height",this.height+margin.top+margin.bottom);

				// let focus = svg.append("g")
				// 				.classed("main-date-chart", true)
				// 				.attr("transform", `translate(${margin.left}, ${margin.top})`)
				// let context = svg.append("g")
				// 				.attr("transform", `translate(${margin2.left}, ${margin2.top})`)
				
				let dataset = Object.values(this.months_data)
				
				var maxHeight=d3.max(dataset,(d) => d)
	            var minHeight=d3.min(dataset,(d) => d)

	            var graph = svg
         
				var bar = graph.selectAll("g")
							.data(dataset)
							.enter()
							.append("g")
							.attr("transform", function(d, i) {
								return "translate(0," + i * maxHeight + ")";
							});
							bar.append("rect").attr("width", function(d) {
								return d * scaleFactor;
							})
							.attr("height", maxHeight - 1);
         
				bar.append("text")
					.attr("x", function(d) { return (d*scaleFactor); })
					.attr("y", maxHeight / 2)
					.attr("dy", ".35em")
					.text(function(d) { return d; });

				
				console.log(dataset)

			},
			renderChart_old () {
				let margin = {top: 20, right: 20, bottom: 90, left: 50}
				let margin2 = {top: this.height - 70, right: 20, bottom: 30, left: 50}
				this.width = this.width - margin.left - margin.right
				this.height = this.height - margin.top - margin.bottom
				let height2 = 50
				let that = this

				if (!d3.select("#date-chart-continer svg").empty()) {
					d3.selectAll("#date-chart-continer svg").remove()
				}

				let svg = d3.select("#date-chart-continer").append("svg")
					// .attr("viewBox", [0, 0, this.width+margin.left+margin.right, this.height+margin.top+margin.bottom])
						.attr("width",this.width+margin.left+margin.right)
						.attr("height",this.height+margin.top+margin.bottom);

				let focus = svg.append("g")
								.classed("main-date-chart", true)
								.attr("transform", `translate(${margin.left}, ${margin.top})`)
				let context = svg.append("g")
								.attr("transform", `translate(${margin2.left}, ${margin2.top})`)




				let dataset = this.observations.map(d => d.observed_on)
				console.log(dataset)


				var maxHeight=d3.max(dataset,(d) => d)
	            var minHeight=d3.min(dataset,(d) => d)

	            this.yScale = d3.scaleLinear().range([0,this.height]).domain([maxHeight*1.1,0])
				this.xScale = d3.scaleBand().rangeRound([0,this.width]).domain(d3.range(1,dataset.length,1)).padding(0.1)
				this.yScale2 = d3.scaleLinear().range([0,height2]).domain([maxHeight*1.1,0])
				this.xScale2 = d3.scaleBand().rangeRound([0,this.width]).domain(d3.range(1,dataset.length,1)).padding(0.1)

				var yAxis = d3.axisLeft(this.yScale).tickSize(-this.width)
				var yAxisGroup = focus.append("g").call(yAxis)
				this.xAxis = d3.axisBottom(this.xScale)
				this.xAxisGroup = focus.append("g").call(this.xAxis).attr("transform", "translate(0,"+this.height+")")

				var xAxis2 = d3.axisBottom(this.xScale2)
				var xAxisGroup2 = context.append("g").call(xAxis2).attr("transform", "translate(0,"+height2+")")

				this.bars1 = focus.selectAll("rect").data(dataset).enter().append("rect")
				this.bars1.attr("x", (d,i) => this.xScale(i)) //i*(width/dataset.length)
						.attr("y", (d) => this.yScale(d)) //for bottom to top
						.attr("width", this.xScale.bandwidth()/*width/dataset.length-barpadding*/)
						.attr("height", (d) =>  this.height - this.yScale(d))

				this.bars1.attr("fill","steelblue")
					.on('mouseover', (d) => that.tooltip.html(`<div>${d} Observations</div>`).style('visibility', 'visible'))
					.on('mousemove', function () {
	  						that.tooltip
	  							.style('top', d3.event.pageY - 10 + 'px')
	  							.style('left', d3.event.pageX + 10 + 'px')
	  						})
					.on('mouseout', () => that.tooltip.html('').style('visibility', 'hidden'))

				var bars2 = context.selectAll("rect").data(dataset).enter().append("rect")
				bars2.attr("x", (d,i) => that.xScale2(i))
					.attr("y",(d) => that.yScale2(d))//for bottom to top
					.attr("width", that.xScale2.bandwidth()/*width/dataset.length-barpadding*/)
						.attr("height", (d) => height2 - that.yScale2(d))
				bars2.attr("fill", "red")

				var brush = d3.brushX()
						.extent([[0,0],[this.width,height2]])//(x0,y0)  (x1,y1)
						.on("brush",this.brushed)//when mouse up, move the selection to the exact tick //start(mouse down), brush(mouse move), end(mouse up)
						.on("end",this.brushend)

				if (this.watch_init_flag && this.selected_dates.length > 0) {
					let extent = [this.xScale2(Math.min(...this.selected_dates)), this.xScale2(Math.max(...this.selected_dates))]
					this.renderBrushedChart(this.selected_dates)
					context.append("g")
						.attr("class","brush")
						.call(brush)
						.call(brush.move,extent);
				} else {
					context.append("g")
						.attr("class","brush")
						.call(brush)
						.call(brush.move,this.xScale2.range());
				}

			},
			brushed () {
				if (!d3.event.sourceEvent) return // Only transition after input.
				if (!d3.event.selection) return // Ignore empty selections.
				if(d3.event.sourceEvent && d3.event.sourceEvent.type === "zoom") return // ignore brush-by-
					//scaleBand of bar chart is not continuous. Thus we cannot use method in line chart.
					//The idea here is to count all the bar chart in the brush area. And reset the domain
				var newInput = []
				var brushArea = d3.event.selection
				let that = this

				if(brushArea === null) brushArea = this.xScale.range()

				this.xScale2.domain().forEach((d) => {
					var pos = that.xScale2(d) + that.xScale2.bandwidth()/2
					if (pos >= brushArea[0] && pos <= brushArea[1]){
					  newInput.push(d)
					}
				})

				this.renderBrushedChart(newInput)
			},
			brushend () {
				if (!d3.event.sourceEvent) return; // Only transition after input.
				if (!d3.event.selection) return; // Ignore empty selections.
				if(d3.event.sourceEvent && d3.event.sourceEvent.type === "zoom") return; // ignore brush-by-
					//scaleBand of bar chart is not continuous. Thus we cannot use method in line chart.
					//The idea here is to count all the bar chart in the brush area. And reset the domain
				var newInput = []
				var brushArea = d3.event.selection
				let that = this
				if(brushArea === null) brushArea = this.xScale.range()

				this.xScale2.domain().forEach((d) => {
					var pos = that.xScale2(d) + that.xScale2.bandwidth()/2;
					if (pos >= brushArea[0] && pos <= brushArea[1]){
						newInput.push(d);
					}
				})

				//relocate the position of brush area
				var increment = 0;
				var left = this.xScale2(d3.min(newInput));
				var right = this.xScale2(d3.max(newInput))+this.xScale2.bandwidth();
				this.emitDate(newInput)
			},
			renderBrushedChart (data){
				this.xScale.domain(data)
				let that = this

				this.bars1.attr("x",(d,i) => that.xScale(i))
						.attr("y",(d) => that.yScale(d))
					.attr("width", this.xScale.bandwidth())//if you want to change the width of bar. Set the width to this.xScale.bandwidth() If you want a fixed width, use this.xScale2.bandwidth(). Note because we use padding() in the scale, we should use this.xScale.bandwidth()
					.attr("height", (d,i) => {
						if(that.xScale.domain().indexOf(i) === -1){
							return 0
						}
						else
							return that.height-that.yScale(d)
					})
				this.xAxisGroup.call(this.xAxis)
			},
			emitDate(d){
				if(this.watch_init_flag == false){
						this.$emit('dateRangeSelected', d)
				}
			}
		}
	}
</script>
