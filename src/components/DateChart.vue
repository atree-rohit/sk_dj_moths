<style>
    #date-chart-continer rect.bar{
        transition: all .6s;
    }
    #date-chart-continer rect.bar:hover{
        fill: red;
    }
</style>

<template>
    <div id="date-chart-continer"></div>
</template>

<script>
import * as d3 from 'd3'

import moment from 'moment'

export default {
    name: 'date-chart',
    props: ['observations'],
    data() {
        return {
            chart: null,
            width: 0,
            height: 0,
        }
    },
    mounted() {
        this.init()
        window.addEventListener('resize', this.init)
    },
    unmounted() {
        window.removeEventListener('resize', this.init)
        d3.selectAll('.tooltip').remove()
    },
    watch: {
        observations() {
            this.init()
        },
    },
    computed: {
        months_data() {
            const op = []
            for (let i = 0; i < 12; i += 1) {
                op.push({
                    month: moment().month(i).format('MMM'),
                    frequency: 0,
                })
            }
            this.observations.forEach((o) => {
                op.forEach((m, i) => {
                    if (m.month === moment(o.observed_on).format('MMM')) {
                        op[i].frequency += 1
                    }
                })
            })
            return op
        },
    },
    methods: {
        BarChart(data, {
            x = (d, i) => i, // given d in data, returns the (ordinal) x-value
            y = (d) => d, // given d in data, returns the (quantitative) y-value
            marginTop = 20, // the top margin, in pixels
            marginRight = 10, // the right margin, in pixels
            marginBottom = 40, // the bottom margin, in pixels
            marginLeft = 25, // the left margin, in pixels
            xDomain, // an array of (ordinal) x-values
            xRange = [marginLeft, this.width - marginRight], // [left, right]
            yDomain, // [ymin, ymax]
            yRange = [this.height - marginBottom, marginTop], // [bottom, top]
            xPadding = 0.1, // amount of x-range to reserve to separate bars
            yLabel, // a label for the y-axis
            color = 'currentColor', // bar fill color
        } = {}) {
            // Compute values.
            const X = d3.map(data, x)
            const Y = d3.map(data, y)

            // Compute default domains, and unique the x-domain.
            if (xDomain === undefined) xDomain = X
            if (yDomain === undefined) yDomain = [0, d3.max(Y)]
            xDomain = new d3.InternSet(xDomain)

            // Omit any data not present in the x-domain.
            const I = d3.range(X.length).filter((i) => xDomain.has(X[i]))

            // Construct scales, axes, and formats.
            const xScale = d3.scaleBand(xDomain, xRange).padding(xPadding)
            const yScale = d3.scaleLinear(yDomain, yRange)
            const xAxis = d3.axisBottom(xScale).tickSizeOuter(0)
            const yAxis = d3.axisLeft(yScale).tickFormat(d3.format('.0f')).ticks(d3.max(Y))

            if (!d3.select('#date-chart-continer svg').empty()) {
                d3.selectAll('#date-chart-continer svg').remove()
            }
            let tooltip = null

            if (!d3.select('.tooltip').empty()) {
                tooltip = d3.select('.tooltip')
            } else {
                tooltip = d3.select('body')
                    .append('div')
                    .attr('class', 'tooltip')
                    .style('opacity', 0)
            }

            const svg = d3.select('#date-chart-continer').append('svg')
                .attr('width', this.width)
                .attr('height', this.height)
                // .attr('viewBox', [0, 0, width, height])
                .attr('style', 'max-width: 100%; height: auto; height: intrinsic;')

            svg.append('g')
                .attr('transform', `translate(${marginLeft},0)`)
                .call(yAxis)
                .call((g) => g.select('.domain').remove())
                .call((g) => g.selectAll('.tick line').clone()
                    .attr('x2', this.width - marginLeft - marginRight)
                    .attr('stroke-opacity', 0.25))
                .call((g) => g.append('text')
                    .attr('x', -marginLeft)
                    .attr('y', 10)
                    .attr('fill', 'currentColor')
                    .attr('text-anchor', 'start')
                    .text(yLabel))

            const mouseover = () => tooltip.transition()
                .duration(200)
                .style('opacity', 0.9)

            const mousemove = (event, d) => tooltip.html(`${X[d]} : ${Y[d]} observation${(Y[d] > 1) ? 's' : ''}`)
                .style('left', `${(event.pageX - 50)}px`)
                .style('top', `${(event.pageY - 10)}px`)

            const mouseout = () => tooltip.transition()
                .duration(500)
                .style('opacity', 0)

            svg.append('g')
                .attr('fill', color)
                .selectAll('rect')
                .data(I)
                .join('rect')
                .attr('x', (i) => xScale(X[i]))
                .attr('y', (i) => yScale(Y[i]))
                .classed('bar', true)
                .attr('height', (i) => yScale(0) - yScale(Y[i]))
                .attr('width', xScale.bandwidth())
                .on('mouseover', mouseover)
                .on('mousemove', (event, d) => mousemove(event, d))
                .on('mouseout', mouseout)

            svg.append('g')
                .attr('transform', `translate(0,${this.height - marginBottom})`)
                .call(xAxis)
                .selectAll('text')
                .style('text-anchor', 'end')
                .attr('dx', '-.8em')
                .attr('dy', '.15em')
                .attr('transform', 'rotate(-65)');
            // return svg.node()
        },
        init() {
            this.width = parseInt(d3.select('#chart').style('width'), 10)
            this.height = parseInt(d3.select('#chart').style('height'), 10)
            this.chart = this.BarChart(this.months_data, {
                x: (d) => d.month,
                y: (d) => d.frequency,
                xDomain: this.months_data.map((d) => d.month),
                yFormat: '',
                yLabel: '',
                color: 'steelblue',
            })
        }
    },
}
</script>
