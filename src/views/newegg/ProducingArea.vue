<template>
	<!--组件:主产区,主销区-->
	<div class="">
		<div class="select-type">
			<tab v-model="$store.state.egg_price.type">
				<tab-item @on-item-click="chanClick()">产区</tab-item>
				<tab-item @on-item-click="xiaoClick()">销区</tab-item>
			</tab>
		</div>
		<div class="line"></div>
	</div>
</template>

<script>
	import { Tab, TabItem } from 'vux'
	import { userAction } from "../../api/api"
	export default {
		components: {
			Tab,
			TabItem
		},
		methods: {
			//这个组件在蛋价和原料价格都出现过,判断下然后确定用户的行为
			chanClick() {
				if(this.$route.name == "PriceChart") {
					userAction({
						ticket: this.$store.state.ticket,
						module: 15151,
						cur_version: this.$store.state.cur_version
					})
				} else {
					userAction({
						ticket: this.$store.state.ticket,
						module: 1511,
						cur_version: this.$store.state.cur_version
					})
				}
				console.log(this.$store.state.egg_price.type)
			},
			xiaoClick() {
				if(this.$route.name == "PriceChart") {
					userAction({
						ticket: this.$store.state.ticket,
						module: 15152,
						cur_version: this.$store.state.cur_version
					})
				} else {
					userAction({
						ticket: this.$store.state.ticket,
						module: 1512,
						cur_version: this.$store.state.cur_version
					})
				}
				console.log(this.$store.state.egg_price.type)
			}
		},
		mounted(){
			console.log(this.$store.state.egg_price.type)
		}
	}
</script>

<style>
	.select-type{
		width: 100%;
		margin: 0;
	}
</style>