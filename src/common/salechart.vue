<template>
	<div class="chart111">
		<div class="drawtwo" id="drawtwo">

		</div>
	</div>
</template>

<script>
	export default {
		props:['xdate','ydate'],
		data() {
			return {
				
			}
		},
		watch: {
			'$store.state.info_sale':'drawLineone'
		},
		mounted() {
			this.drawLineone();
		},
		methods: {
			drawLineone() {
				this.$emit('getdata')
				let myChart = this.$echarts.init(document.getElementById('drawtwo'));
				myChart.setOption({
					grid: {
						top: 40,
						bottom: 30,
						left: 45,
						right: 10
					},
					tooltip:{
						trigger:'axis',
					},
					legend: {
						show: true,
						data: ['鸡蛋价格'],
						textStyle: {
							color: '#999',
						},
						top: '10',
						right: '0',
						itemGap: 1,
						itemWidth: 6,
						itemHeight: 6,
					},
					xAxis: {
						type: 'category',
						data: this.xdate,
						nameTextStyle: {
							color: "#999"
						},
						axisLine: {
							show: false
						},
						axisTick: {
							show: false
						},
						axisLabel: {
							color: '#999'
						}
					},
					yAxis: {
						type: 'value',
						name: '单位（元 / 斤）',
						nameLocation: 'end',
						nameTextStyle: {
							color: "#999"
						},
						axisLine: {
							show: false
						},
						axisTick: {
							show: false
						},
						axisLabel: {
							color: '#999'
						},
						scale: true,
						splitLine: {
							lineStyle: {
								color: '#eee'
							}
						}
					},
					series: [{
						data: this.ydate,
						type: 'line',
						name: '鸡蛋价格',
						symbol: 'circle',
						symbolSize: 5, //图标尺寸
						lineStyle: {
							normal: {
								color: "#57b663" //连线颜色
							}
						},
						itemStyle: {
							normal: {
								color: "#57b663", //图标颜色
							}
						},

					}]
				})
			}
		}
	}
</script>

<style>
	.drawtwo {
		width: 100%;
		height: 20rem;
	}
</style>