<template>
	<div class="egg-price">
		<search v-if="egg_chart" placeholder="请输入您要查询的地区" @on-change="getResult" cancel-text=" " @on-focus="onFocus" @on-cancel="is_search_focus = false" @result-click="resultClick" :results="results" v-model="search_text" position="absolute" auto-scroll-to-top top="46px" @on-submit="onSearch" ref="search"></search>
		<div class="egg-price-search" style="height: 44px" v-show="is_search_focus"></div>
		<producing-area></producing-area>
		<sale-address></sale-address>
		<p class="today-egg" style="margin-top: 20px;">今日蛋价</p>
		<div class="update-time">最后更新时间： {{ update_time }}</div>

		<x-table :cell-bordered="false">
			<thead>
				<tr class="theme-bg">
					<th>地区</th>
					<th>鸡蛋价格(元/斤)</th>
					<th>涨跌幅(元/斤)</th>
				</tr>
			</thead>
			<tbody>
				<template v-for="(data,index) in price_data">
					<tr>
						<td>{{ data.name }}</td>
						<td>{{ data.price }}</td>
						<td><span :class="data.gains>0?'theme-color-red':'theme-color'" :style="{color:data.gains == 0?'#333':false}">{{ data.gains.toFixed(2) }}</span></td>
					</tr>
				</template>
			</tbody>
		</x-table>

		<div class="h60">

		</div>

		<div class="tab fix-box">
			<router-link class="tab-item" to="/p/price-forecast"> <span @click="yuceClick">蛋价预测</span></router-link>
			<router-link class="tab-item selected" to="/p/price-chart"><span @click="eggEarchtClick">查看蛋价曲线</span></router-link>
		</div>
		<div class="mask" v-if="is_search_focus"></div>
	</div>
</template>

<script>
	import { Search, Group, Cell, XButton, XTable, LoadMore, ButtonTab, ButtonTabItem, XAddress, ChinaAddressV3Data, dateFormat, Popup } from 'vux'
	import SaleAddress from '../views/eggprice/SaleAddress'
	import ProducingArea from '../views/eggprice/ProducingArea'
	import 'vue-awesome/icons/map-marker'
	import Icon from 'vue-awesome/components/Icon'
	import { userAction } from '../api/api'
	import { search, getSaleSearch, sreachArea } from '../api/EggPrice'
	//匹配汉字
	var regZH = new RegExp("[\\u4E00-\\u9FFF]+", "g");
	export default {
		components: {
			Search,
			Group,
			Cell,
			XButton,
			XTable,
			LoadMore,
			ButtonTab,
			XAddress,
			Icon,
			ProducingArea,
			SaleAddress,
			ButtonTabItem,
			Popup
		},
		data() {
			return {
				price_data: [],
				results: [{
					title: ''
				}],
				search_text: '',
				is_search_focus: false, //因app不需要展示header,在下面会做判断
				update_time: '',
				egg_chart: true, //因app不需要展示搜索框,在下面会做判断
				city_name: ''
			}
		},
		mounted() {
			this.$store.state.header.title = '鸡蛋价格查询';
			//添加那个绿色的小三角
			var xheader = document.querySelector(".vux-header-title");
			xheader.classList.add('title-after');

			this.$store.state.price_type = 0;
			this.init();
			//判断是不是在蛋鸡管家app中
			var ua = navigator.userAgent.toLowerCase();
			if(ua.indexOf('djgj_jn') > -1) {
				//如果在app中 那么header 搜索框 都隐藏
				this.$store.state.header.is_show_header = false;
				this.egg_chart = false;
				if(this.$route.query.ticket != undefined) this.$store.state.ticket = this.$route.query.ticket;
			}
			userAction({
				ticket: this.$store.state.ticket,
				module: 154,
				cur_version: this.$store.state.cur_version
			})
		},
		watch: {
			'$store.state.egg_price': {
				handler: function() {
					this.init();
				},
				deep: true
			}
		},
		methods: {
			init() {
				this.$vux.loading.show({
					text: '数据加载中...'
				})
				//获取当前时间
				var d = new Date();
				var str = d.getFullYear() + "-" + (d.getMonth() + 1) + "-" + d.getDate();
				this.price_data = [];
				//判断是主产区还是主销区
				if(this.$store.state.egg_price.type == 0) {
					search({
						area: this.$store.state.egg_price.area
					}).then(rs => {
						this.price_data = rs.data.search;
						this.update_time = rs.data.saveTime;
						this.$vux.loading.hide();
					})
				} else {
					getSaleSearch({
						area: this.$store.state.egg_price.area
					}).then(rs => {
						this.price_data = rs.data;
						if(typeof(rs.data) != 'undefined') {
							if(rs.data[0].savetime != 'undefined') {
								this.update_time = rs.data[0].savetime;
							}
						}
						this.$vux.loading.hide();
					})
				}
				//微信分享
				this.$wechat.ready(() => {
					//					var title = str + this.$store.state.egg_price.area + '鸡蛋、原料、淘汰鸡价格';
					var title = str + '全国各地鸡蛋、原料、淘汰鸡价格';
					var link = 'https://m.danjiguanjia.com/#/p/egg-price';
					var imgUrl = 'https://api.danjiguanjia.com/public/icon120.png';
					this.$wechat.onMenuShareTimeline({
						title: title,
						link: link,
						imgUrl: imgUrl
					});
					this.$wechat.onMenuShareAppMessage({
						title: title,
						desc: '天天查价格,掌握最新价格行情信息',
						link: link,
						imgUrl: imgUrl
					});
					this.$wechat.onMenuShareQQ({
						title: title,
						link: link,
						imgUrl: imgUrl
					});
					this.$wechat.onMenuShareWeibo({
						title: title,
						link: link,
						imgUrl: imgUrl
					});
				});

				console.log(this.$store.state.egg_price.area)
			},
			getResult() {
				//只有汉字才能搜索
				if(regZH.test(this.search_text)) {
					sreachArea({
						area: this.search_text
					}).then(rs => {
						if(rs.data.length > 0) {
							this.results = [];
							for(var i = 0; i < rs.data.length; i++) this.results.push({
								title: rs.data[i]
							});
						}
					})
				}
			},
			resultClick(item) {
				this.title = item.title;
				this.is_search_focus = false;
				//跳转搜索结果页面
//				this.$router.push('/p/search-result/' + item.title);
				this.$router.push('/p/search-result/' + item.title);
			},
			onFocus() {
				this.is_search_focus = true;
				var xheader = document.querySelector(".vux-header-title");
				xheader.classList.remove('title-after');
				sreachArea().then(rs => {
					if(rs.data.length > 0) {
						this.results = [];
						for(var i = 0; i < rs.data.length; i++) this.results.push({
							title: rs.data[i]
						});
					}
				})
			},
			onSearch() {
				this.$refs.search.setBlur();
			},
			logHide(str) {
				//console.log('on-hide', str)
			},
			logShow(str) {
				//console.log('on-show')
			},
			yuceClick() {
				userAction({
					ticket: this.$store.state.ticket,
					module: 1514,
					cur_version: this.$store.state.cur_version
				})
			},
			eggEarchtClick() {
				userAction({
					ticket: this.$store.state.ticket,
					module: 1515,
					cur_version: this.$store.state.cur_version
				})
			}
		}
	}
</script>

<style lang="less">
	.vux-search-box {
		height: 100%;
	}
	
	.today-egg {
		text-align: center;
		color: #57b663;
		font-size: 1rem;
		padding-bottom: 0.5rem;
	}
	
	.vux-tap-active {
		width: 90%;
		background-color: #FFFFFF;
	}
	
	.select-type {
		width: 90%;
		margin: 15px auto;
	}
	
	.update-time {
		margin-bottom: 15px;
		text-align: center;
	}
	
	.egg-price {
		background: #FFFFFF;
		.h60 {
			height: 60px;
		}
		.fix-box {
			position: fixed;
			bottom: -2px;
			width: 100%;
			overflow: hidden;
		}
		.tab {
			.tab-item {
				color: #56b563;
				border-top: 1px solid #ddd;
				width: 50%;
				text-align: center;
				line-height: 50px;
				height: 50px;
				float: left;
				background: #fff;
				font-size: 14px;
			}
			.tab-item.selected {
				color: #fff;
				background: #56b563;
				border-top: none;
			}
		}
		.weui-search-bar__box {
			padding: 2px 30px;
		}
		.weui-search-bar__box .weui-icon-search {
			top: 2px;
		}
		.weui-search-bar__label {
			top: 5.5px;
		}
		.mask {
			width: 100%;
			position: absolute;
			top: 0;
			left: 0;
			height: 150vh;
			background: black;
			filter: alpha(Opacity=80);
			-moz-opacity: 0.5;
			opacity: 0.5;
		}
	}
</style>