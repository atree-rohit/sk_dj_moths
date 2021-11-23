<style>
	#map-container svg{
		height: 33vh;
		width: 19.75vw;
	}
	.map-boundary > path {
		fill: white;
		stroke: rgba(75,0,0,.5);
		stroke-width: 1px;
	}
	.poly_text{
		fill: #545;
		font-size: 0.5vw;
		transition: fill,text-shadow .125s;
	}
	.poly_text:hover{
		fill: #00c;
		text-shadow: 0px 0px 5px #fff;
		cursor: pointer;
		font-weight: 1000;
	}
	@media screen and (max-width: 800px) {
		.poly_text{
			font-size: 3.5vw;
		}
		#map-container svg{
			height: 100%;
			width: 100%;
		}
	}

</style>

<template>
	<div id="map-container"></div>
</template>

<script>
import * as d3 from "d3"
import * as d3Legend from "d3-svg-legend"
import districts_map from '../assets/book_data/districts_map.json'

export default {
	name:"SpeciesMap",
	props: [ "observations" ],
	data() {
		return{
			states: null,
			path: null,
			svg: {},
			projection: {},
			colors: {},
			legend: {},

			state_data: {},
			selected:"All",
			state_max: 0,
			height: "100" ,
			width: "100",
			// tooltip:this.popup,
			map_first_render:true,
		}
	},
	mounted(){
		this.init()
	},
	computed:{
		points () {
			let op = []
			this.observations.forEach(o => {
				let l = o.location.split(",")
				// op.push({
				// 	id: o.id,
				// 	name: o.place_guess,
				// 	latitude: l[0],
				// 	longitude: l[1]
				// })
				op.push([l[1], l[0], o.id, o.place_guess])
			})
			return op
		},

		zoom() {
			let that = this
			return d3.zoom(event)
					.scaleExtent([.5, 50])
					.translateExtent([[-0.5 * this.width,-0.75 * this.height],[2.5 * this.width, 2.5 * this.height]])
					.on('zoom', function() {
						that.svg.selectAll('.poly_text')
							.attr('transform', event.transform),
						that.svg.selectAll('path')
							.attr('transform', event.transform),
						that.svg.selectAll('circle')
							.attr('transform', event.transform)
						// .attr("r", 2 / event.transform.k)
					})
		},
	},
	watch: {
	},
	methods:{
		init () {
			this.renderMap()
		},
		renderMap () {
			if (!d3.select("#map-container svg").empty()) {
				d3.selectAll("#map-container svg").remove()
			}

			this.svg = d3.select("#map-container")
						.append("svg")
							.attr("preserveAspectRatio", "xMinYMin meet")
							// .attr("width", this.width)
							// .attr("height", this.height)
							.attr("viewbox", `0 0 ${this.width} ${this.height}`)
							.style("background-color", "rgb(190, 229, 235)")
							.classed("svg-content d-flex m-auto", true)

			this.projection = d3.geoMercator().scale(6500).center([91, 26.25])
			this.path = d3.geoPath().projection(this.projection)

			let base = this.svg.append("g")
						.classed("map-boundary", true)

			let base_text = base.selectAll("text").append("g")
			base = base.selectAll("path").append("g")
			this.states = base.append("g").classed("states", true)
			let that = this

			districts_map.features.forEach( district => {
				let s_name = district.properties.district
				let that = this

				let current_state = this.states.append("g")
					.data([district])
					.enter().append("path")
					.attr("d", this.path)
					.attr("id", this.stateID(s_name))
					.attr("title", s_name)
			})
			let map_points = this.svg.append('g')
				.classed('map-points', true)
				.selectAll("circle")
				.data(this.points).enter()
				.append("circle")
				.attr("cx", (d) => this.projection(d)[0])
				.attr("cy", (d) => this.projection(d)[1])
				.attr("r", "5px")
				.attr("title", (d) => d)
				.attr("stroke", "red")
				.attr("fill", "rgba(255,0,0,.25)")

			// this.svg.call(this.zoom)

		},
		init_old () {
			this.map_first_render = true
			this.states = null
			this.path = null
			this.svg = {}
			this.projection = {}
			this.colors = {}
			this.legend = {}


			this.state_data = {}
			this.state_max = 0
			this.height = window.innerHeight * 0.85
			this.width = window.innerWidth * 0.5
			if(window.innerWidth < 800){
				this.height = window.innerHeight * 0.5
				this.width = window.innerWidth * 0.9
			}
			districts_map.features.forEach(s => {
				this.state_data[s.properties.ST_NM] = [];
			})

			this.map_data.forEach(o => {
				if(Object.keys(this.state_data).indexOf(o.state) != -1){
					this.state_data[o.state].push(o)
				} else {
					console.log("unmatched state name", o.state, o)
				}
			});

			Object.keys(this.state_data).forEach(s => {
				if(this.state_data[s].length > this.state_max)
				this.state_max = this.state_data[s].length;
			})
			this.colors = d3.scaleLinear()
				.domain([0, 1, this.state_max*.25, this.state_max])
				.range(["#f77", "#ca0", "#ada", "#3d3"])

			this.legend = d3Legend.legendColor()
								.shapeWidth(45)
								.scale(this.colors)
								.labelFormat(d3.format(".0f"))
								.orient('horizontal')
								.labelOffset(-10)
								.labelAlign("start")
								.cells(6)
								// .shapePadding(47)

			this.renderMap()
			this.clicked(this.selectedGeoJson)
			this.map_first_render = false
		},
		renderMap_old () {
			this.selected = this.selected_state

			if (!d3.select("#map-container svg").empty()) {
				d3.selectAll("#map-container svg").remove()
			}
			this.svg = d3.select("#map-container")
						.append("svg")
							.attr("preserveAspectRatio", "xMinYMin meet")
							.attr("width", this.width)
							.attr("height", this.height)
							.style("background-color", "rgb(190, 229, 235)")
							.classed("svg-content d-flex m-auto", true)

			this.projection = d3.geoMercator().scale(850).center([87, 25.5])
			this.path = d3.geoPath().projection(this.projection)


			if(this.height > this.width){
				this.legend.shapeWidth(35)
				.cells(4)
				// .shapePadding(37)
			}


			let base = this.svg.append("g")
			.classed("map-boundary", true)

			let base_text = base.selectAll("text").append("g")
			base = base.selectAll("path").append("g")
			this.states = base.append("g").classed("states", true)
			let that = this

			districts_map.features.forEach(state=> {
				let s_name = state.properties.ST_NM
				let that = this

				let current_state = this.states.append("g")
				.data([state])
				.enter().append("path")
				.attr("d", this.path)
				.attr("id", this.stateID(s_name))
				.attr("title", s_name)
				// .on('mouseover', function (d, i) {
				// 	that.tooltip.html(
				// 		`<table>
				// 		<tr><td>State</td><td>${s_name}</td></tr>
				// 		<tr><td>Observations</td><td>${that.stateStats[s_name].observations}</td></tr>
				// 		<tr><td>Users</td><td>${that.stateStats[s_name].users.size}</td></tr>
				// 		<tr><td>Unique Taxa</td><td>${that.stateStats[s_name].species.size}</td></tr>
				// 		</table>`)
				// 		.style('visibility', 'visible');
				// 	})
				// .on('mousemove', function () {
				// 	that.tooltip
				// 	.style('top', d3.event.pageY - 10 + 'px')
				// 	.style('left', d3.event.pageX + 10 + 'px');
				// })
				// .on('mouseout', () => that.tooltip.html(``).style('visibility', 'hidden'))
				.on("click", this.clicked)


				if(this.stateData[s_name] == undefined){
					current_state.attr("fill", (d) => colors(-1))
				} else if (s_name == this.selected) {
					current_state.classed("state-selected", true)
				} else {
					current_state.attr("fill", (d) => this.colors(this.stateData[s_name].length))
				}

			})
			if(this.selected == "All"){
				districts_map.features.forEach(state=> {
					let s_name = state.properties.ST_NM
					let label = base_text.append("g")
						.data([state])
						.enter().append("text")
						.classed("poly_text", true)
						.attr("x", (h) => this.path.centroid(h)[0] )
						.attr("y", (h) => this.path.centroid(h)[1] )
						.attr("text-anchor", "middle")
						.text(this.stateData[s_name].length)
						// .on('mouseover', function (d, i) {
						// 	that.tooltip.html(
						// 		`<table>
						// 		<tr><td>State</td><td>${s_name}</td></tr>
						// 		<tr><td>Observations</td><td>${that.stateStats[s_name].observations}</td></tr>
						// 		<tr><td>Users</td><td>${that.stateStats[s_name].users.size}</td></tr>
						// 		<tr><td>Unique Taxa</td><td>${that.stateStats[s_name].species.size}</td></tr>
						// 		</table>`)
						// 		.style('visibility', 'visible');
						// 	})
						// .on('mousemove', function () {
						// 	that.tooltip
						// 	.style('top', d3.event.pageY - 10 + 'px')
						// 	.style('left', d3.event.pageX + 10 + 'px');
						// })
						// .on('mouseout', () => that.tooltip.html(``).style('visibility', 'hidden'))
						.on("click", this.clicked)
				})
			}

			this.svg.append("g")
				.attr("transform", "translate("+this.width*.5+", 50)")
				.call(this.legend)
				// .append("text")
				// .classed("map_label", true)
				// .attr("dx", 5)
				// .attr("dy", -10)
				// .classed("h1", true)
				// .text(this.selected)

			this.svg.call(this.zoom)
			this.mapPoints()

			// if(this.map_first_render){
			// 	this.clicked(this.selectedGeoJson)
			// 	this.map_first_render = false

			// }
		},
		stateID(s){
			return s.replaceAll(" ", "_").replaceAll("&", "")
		},
		clicked(d) {
			this.tooltip.html(``).style('visibility', 'hidden')
			let state = d.properties.ST_NM

			if(state == this.selected && state != 'All')
				if (!d3.select("#map-container .poly_text").empty()) {
					d3.selectAll("#map-container .poly_text").remove()
				}
			let [[x0, y0], [x1, y1]] = [[0,0],[0,0]]

			this.states.transition().style("fill", null)

			if(d3.select(".state-selected")["_groups"][0][0] != null){
				d3.select("#" + d3.select(".state-selected")["_groups"][0][0].id).attr("class", null)

			}

			if(this.map_first_render){
				if(state == "All"){
					[[x0, y0], [x1, y1]] = this.path.bounds(districts_map)
				} else {
					[[x0, y0], [x1, y1]] = this.path.bounds(d)
					d3.select("#" + this.stateID(state)).classed("state-selected", true)
				}
			} else {
				if(this.selected == state){
					[[x0, y0], [x1, y1]] = this.path.bounds(districts_map)
				} else {
					[[x0, y0], [x1, y1]] = this.path.bounds(d)
					d3.select("#" + this.stateID(state)).classed("state-selected", true)
				}
				if(this.selected == state){
					this.$emit('stateSelected', 'All')
				} else {
					this.$emit('stateSelected', state)
				}
			}

			this.svg.transition().duration(750).call(
				this.zoom.transform,
				d3.zoomIdentity
				.translate(this.width / 2, this.height / 2)
				.scale(Math.min(8, 0.9 / Math.max((x1 - x0) / this.width, (y1 - y0) / this.height)))
				.translate(-(x0 + x1) / 2, -(y0 + y1) / 2),
			)
			/*
			*/
		},
		mapPoints(){
			let points = []

			if(this.selected != 'All'){
				this.state_data[this.selected].forEach(o => {
					let coords = o.location.split(",")
					points.push([coords[1], coords[0], o.id, o.place_guess]);
				})
			}

			if(points.length > 0){
				let map_points = this.svg.append('g')
				.classed('map-points', true)
				.selectAll("circle")
				.data(points).enter()
				.append("circle")
				.attr("cx", (d) => this.projection(d)[0])
				.attr("cy", (d) => this.projection(d)[1])
				.attr("r", "0px")

				// map_points.on("click", (d) => that.setMissingState(d))
			}
		},
	}
};
</script>