<template>
	<svg id="my-svg" v-bind:width="panelWidth" v-bind:height="panelHeight">
		<g v-for="(item, index) in data" class="bar" shape-rendering="crispEdges"
			v-bind:class="[{ hidden: index === m || index === n }, { peek: peeking === index }, { assign: assigning === index }]">
			<rect v-bind="{ 'x':index*barWidth, 'y':panelHeight-item*heightRatio }" v-bind:width="barWidth" v-bind:height="item*heightRatio"/>
    	</g>
		<g v-if="m >= 0" class="bar selected" shape-rendering="crispEdges">
			<rect v-bind="{ 'x':(m+diff)*barWidth, 'y':panelHeight-data[m]*heightRatio }" v-bind:width="barWidth" v-bind:height="data[m]*heightRatio"/>
    	</g>
		<g v-if="n >= 0" class="bar selected" shape-rendering="crispEdges">
			<rect v-bind="{ 'x':(n-diff)*barWidth, 'y':panelHeight-data[n]*heightRatio }" v-bind:width="barWidth" v-bind:height="data[n]*heightRatio"/>
    	</g>
	</svg>
</template>

<script>
import eventBus from '@/js/EventBus'
export default {
	data () {
	    return {
			data: [],
			panelWidth: 1,
			panelHeight: 1,
			maxHeight: 1,
			m: -1,
			n: -1,
			peeking: -1,
			assigning: -1,
			diff: 0,
			step: 1,
			sorting: false,
			speed: 1000
		}
	},
	methods: {
		onResize(event) {
			let mySvg = document.querySelector('#my-svg')
			this.panelWidth = mySvg.parentNode.clientWidth
			this.panelHeight = mySvg.parentNode.clientHeight
			this.maxHeight = Math.max(...this.data)
		},
		async shuffle() {
			if (this.sorting) {
				alert('Can\'t shuffle while sorting')
				return
			}
			this.sorting = true
			let array = this.data
			let currentIndex = array.length-1, randomIndex
			while (currentIndex > -1) {
				randomIndex = (Math.random() * array.length)|0
				await this.swap(randomIndex, currentIndex)
				currentIndex -= 1
			}
			let _this = this
			return new Promise((resolve) => {
				_this.sorting = false
				resolve(0)
			})
			this.data = array
			this.m = -2 // change something in data to trigger vue to reload graph
			this.m = -1
			promise.then(() => { return })
		},
		shuffleInst() {
			if (this.sorting) {
				alert('Can\'t shuffle while sorting')
				return
			}
			let array = this.data
			let currentIndex = array.length-1, randomIndex
			let promise = new Promise((resolve) => {
				while (currentIndex > -1) {
					randomIndex = (Math.random() * array.length)|0;
					[array[currentIndex], array[randomIndex]] = [array[randomIndex], array[currentIndex]]
					currentIndex -= 1
				}
				resolve(0)
			})
			this.data = array
			this.m = -2 // change something in data to trigger vue to reload graph
			this.m = -1
			promise.then(() => { return })
		},
		swap(m, n) {
			this.m = Math.min(m, n)
			this.n = Math.max(m, n)
			let _this = this
			let done = Math.abs(_this.m - _this.n)
			let step = done/_this.step
			return new Promise((resolve) => {
				if (_this.diff+step >= done) {
					[_this.data[_this.m], _this.data[_this.n]] = [_this.data[_this.n], _this.data[_this.m]]
					_this.m = -1
					_this.n = -1
					_this.diff = 0
					setTimeout(() => {
						resolve(0)
					}, 1000/_this.speed)
				} else {
					let interval = setInterval(() => {
						if (_this.diff < done) _this.diff = Math.min(_this.diff+step, done)
						else {
							[_this.data[_this.m], _this.data[_this.n]] = [_this.data[_this.n], _this.data[_this.m]]
							_this.m = -1
							_this.n = -1
							_this.diff = 0
							clearInterval(interval)
							resolve(0)
						}
					}, 1000/_this.speed)
				}
			})
		},
		peekAt(n) {
			this.peeking = n
			let _this = this
			return new Promise(resolve => {
				setTimeout(() => {
					resolve(_this.data[_this.peeking])
				}, 1000/(_this.speed*2))
			})
		},
		assignAt(i, n) {
			this.data[i] = n
			this.assigning = i
			let _this = this
			return new Promise(resolve => {
				setTimeout(() => {
					resolve(0)
				}, 1000/(_this.speed*2))
			})
		}
	},
	computed: {
		barWidth() {
			return this.panelWidth/this.data.length
		},
		heightRatio() {
			return this.panelHeight/this.maxHeight
		}
	},
	mounted() {
		for (let i = 1; i <= 100; i++) {
			this.data.push(i)
		}
		window.addEventListener('resize', this.onResize)
		this.onResize() // call onResize to make the bars right size
	},
	created() {
		eventBus.$on('sort', id => {
			this.sorting = true
		})
		eventBus.$on('sorted', id => {
			this.sorting = false
			this.peeking = -1
			this.assigning = -1
		})
		eventBus.$on('generateData', data => {
			if (this.sorting) {
				alert('Can\'t change data while sorting')
				return
			}
			this.data = data
			this.onResize() // call onResize to make the bars right size
		})
	},
	beforeDestroy() {
		window.removeEventListener('resize', this.onResize)
	}
}
</script>

<style scoped>
	.bar { fill: gold }
	.selected { fill: lightskyblue }
	.hidden { fill: transparent }
	.peek { fill: tomato }
	.assign { fill: #39FF00 }
	#my-svg { background-color: white; }
</style>
