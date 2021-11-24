<style>
    .tick text{
        font-size: 1.5em !important;
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
    mounted() {
        this.init()
        window.addEventListener('resize', this.windowResize)
    },
    unmounted() {
        window.removeEventListener('resize', this.windowResize)
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
    watch: {
    },
    methods: {
        windowResize(e) {
            this.init()
            console.log(this.width, this.height, e)
        },
        BarChart(data, {
            x = (d, i) => i, // given d in data, returns the (ordinal) x-value
            y = (d) => d, // given d in data, returns the (quantitative) y-value
            title, // given d in data, returns the title text
            marginTop = 20, // the top margin, in pixels
            marginRight = 0, // the right margin, in pixels
            marginBottom = 30, // the bottom margin, in pixels
            marginLeft = 40, // the left margin, in pixels
            xDomain, // an array of (ordinal) x-values
            xRange = [marginLeft, this.width - marginRight], // [left, right]
            yType = d3.scaleLinear, // y-scale type
            yDomain, // [ymin, ymax]
            yRange = [this.height - marginBottom, marginTop], // [bottom, top]
            xPadding = 0.1, // amount of x-range to reserve to separate bars
            yFormat, // a format specifier string for the y-axis
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
            const yScale = yType(yDomain, yRange)
            const xAxis = d3.axisBottom(xScale).tickSizeOuter(0)
            const yAxis = d3.axisLeft(yScale).tickFormat(d3.format('.0f')).ticks(d3.max(Y))

            // Compute titles.
            if (title === undefined) {
                const formatValue = yScale.tickFormat(100, yFormat)
                title = (i) => `${X[i]}\n${formatValue(Y[i])}`
            } else {
                const O = d3.map(data, (d) => d)
                const T = title
                title = (i) => T(O[i], i, data)
            }

            // const svg = d3.create("svg")
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
                    .attr('stroke-opacity', 0.1))
                .call((g) => g.append('text')
                    .attr('x', -marginLeft)
                    .attr('y', 10)
                    .attr('fill', 'currentColor')
                    .attr('text-anchor', 'start')
                    .text(yLabel))

            const bar = svg.append('g')
                .attr('fill', color)
                .selectAll('rect')
                .data(I)
                .join('rect')
                .attr('x', (i) => xScale(X[i]))
                .attr('y', (i) => yScale(Y[i]))
                .attr('height', (i) => yScale(0) - yScale(Y[i]))
                .attr('width', xScale.bandwidth())

            if (title) {
                bar.append('title')
                    .text(title)
            }

            svg.append('g')
                .attr('transform', `translate(0,${this.height - marginBottom})`)
                .call(xAxis)
            // return svg.node()
        },
        init() {
            this.width = parseInt(d3.select('#chart').style('width'), 10)
            this.height = parseInt(d3.select('#chart').style('height'), 10)
            this.chart = this.BarChart(this.months_data, {
                x: (d) => d.month,
                y: (d) => d.frequency,
                xDomain: this.months_data.map((d) => d.month),
                yFormat: '%',
                yLabel: '',
                color: 'steelblue',
            })
        },
    },
}
</script>
