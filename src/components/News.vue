<template>
	<!--热点资讯首页-->
	<div>
		<div class="news_tab">
			<tab :line-width=2 active-color='#57b663'>
				<tab-item @on-item-click="onItemClick" :selected="index == $store.state.news_selected_categroy" v-for="(item,index) in cate_arr">{{ item }}</tab-item>
			</tab>
		</div>

		<div class="news-warp">
			<div class="news">
				<div class="news-list" v-for="(news,index) in news_list">
					<div class="new-list-main" @click="newsClick(news.cms_id)">
						<template v-if="news.video_url!=''">
							<p class="new-list-main-title">{{news.cms_title}}</p>
							<video width="100%" :src="news.video_url" :poster="news.cms_pic" webkit-playsinline="true" x5-video-player-type="h5" controls="controls"></video>
							<ul>
								<li><span class="new-list-main-top" v-if="news.is_top==1">置顶</span></li>
								<li class="new-list-main-li2">{{news.author}}</li>
								<li class="new-list-main-li3"><img src="../assets/images/eye.png" alt="" /> {{news.view}}</li>
								<li class="new-list-main-li4">{{news.create_time}}</li>
							</ul>
						</template>
						<template v-else-if="news.content_img.length>2 ">
							<p class="new-list-main-title">{{news.cms_title}}</p>
							<div class="new-list-main-img">
								<img :src="cov" alt="" v-for="(cov,index) in news.content_img" v-if="index<3" />
							</div>
							<ul>
								<li><span class="new-list-main-top" v-if="news.is_top==1">置顶</span></li>
								<li class="new-list-main-li2">{{news.author}}</li>
								<li class="new-list-main-li3"><img src="../assets/images/eye.png" alt="" /> {{news.view}}</li>
								<li class="new-list-main-li4">{{news.create_time}}</li>
							</ul>
						</template>
						<template v-else>
							<div class="new-list-main-three">
								<template v-if="news.content_img.length>0">
									<div class="new-list-main-left">
										<p class="new-list-main-title title-height">{{news.cms_title}}</p>
										<ul>
											<li><span class="new-list-main-top" v-if="news.is_top==1">置顶</span></li>
											<li class="new-list-main-li2">{{news.author}}</li>
											<li class="new-list-main-li3"><img src="../assets/images/eye.png" alt="" /> {{news.view}}</li>
											<li class="new-list-main-li4">{{news.create_time}}</li>
										</ul>
									</div>
									<div class="new-list-main-right">
										<img :src="news.content_img[0]" alt="" />
									</div>
								</template>
								<template v-else>
									<div class="new-list-main-left-break">
										<p class="new-list-main-title">{{news.cms_title}}</p>
										<ul>
											<li><span class="new-list-main-top" v-if="news.is_top==1">置顶</span></li>
											<li class="new-list-main-li2">{{news.author}}</li>
											<li class="new-list-main-li3"><img src="../assets/images/eye.png" alt="" /> {{news.view}}</li>
											<li class="new-list-main-li4">{{news.create_time}}</li>
										</ul>
									</div>
								</template>
							</div>
						</template>
						<hr/>
					</div>
				</div>
			</div>
		</div>
		<!--无限加载-->
		<mugen-scroll :handler="fetchData" :should-handle="!loading" class="loadings">
			loading...
		</mugen-scroll>

	</div>
</template>
<script>
	import { Tab, TabItem, Sticky, Divider, XButton, Swiper, SwiperItem, Scroller, Spinner } from 'vux'
	import { getWeixinSign, newsData, userAction } from "../api/api"
	import wx from 'weixin-js-sdk';
	import MugenScroll from 'vue-mugen-scroll'
	export default {
		components: {
			Tab,
			TabItem,
			Sticky,
			Divider,
			XButton,
			Swiper,
			SwiperItem,
			Scroller,
			Spinner,
			MugenScroll
		},
		data() {
			return {
				news_list: [],
				item: "",
				page: 0,
				cate: [2, 107, 100, 78],
				cate_arr: ['新闻动态', '政策头条', '养殖宝典', '微周报'],
				category: 0,
				loading: false,
				mustLogin: true
			}
		},
		mounted() {
			this.$store.state.header.title = '热点资讯';
			this.$store.state.header.right_title = "";
			//首次填页面数据
			this.category = this.cate[this.$store.state.news_selected_categroy];
			newsData({
				ticket: this.$store.state.ticket,
				category: this.category
			}).then(res => {
				this.news_list = res.data.data;
			});

			this.$wechat.ready(() => {
				var title = '蛋鸡管家 - 全球蛋鸡产业信息化平台';
				var link = window.location.href;
				var imgUrl = 'https://api.danjiguanjia.com/public/icon120.png';
				this.$wechat.onMenuShareTimeline({
					title: title,
					link: link,
					imgUrl: imgUrl
				});
				this.$wechat.onMenuShareAppMessage({
					title: title,
					desc: '关注蛋鸡管家,随时掌握最新行情信息!',
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

		},
		methods: {
			//tab-item的点击事件,每次点击之后先重置到顶端
			onItemClick(index) {
				this.page = 1;
				this.news_list = [];
				this.category = this.cate[index];
				this.$store.state.news_selected_categroy = index;
				newsData({
					ticket: this.$store.state.ticket,
					category: this.category,
					page: this.page
				}).then(res => {
					this.news_list = res.data.data;
				});
				//用户行为模块
				if(index == 0) {
					userAction({
						ticket: this.$store.state.ticket,
						module: 111,
						cur_version: this.$store.state.cur_version
					})
				}else if(index == 1){
					userAction({
						ticket: this.$store.state.ticket,
						module: 112,
						cur_version: this.$store.state.cur_version
					})
				}else if(index == 2){
					userAction({
						ticket: this.$store.state.ticket,
						module: 113,
						cur_version: this.$store.state.cur_version
					})
				}else{
					userAction({
						ticket: this.$store.state.ticket,
						module: 114,
						cur_version: this.$store.state.cur_version
					})
				}
			},
			//无限加载
			fetchData() {
				this.page++;
				newsData({
					ticket: this.$store.state.ticket,
					category: this.category,
					page: this.page
				}).then(res => {
					for(var i = 0; i < res.data.data.length; i++) {
						this.news_list.push(res.data.data[i]);
					}
				});
			},
			newsClick(id) {
				this.$router.push('/p/news/detail/' + id)
			},
		}
	}
</script>

<style lang="less" scoped>
	.praised {
		color: #FF0000;
	}
	
	.news-warp {
		margin-top: 47px;
	}
	
	.news_tab {
		position: fixed;
		width: 100%;
		z-index: 10;
		top: 45px;
		left: 0;
	}
	
	.loadings {
		width: 90%;
		margin: 0 auto;
		text-align: center;
	}
	
	.news-list:before,
	.news-list:after,
	.data:after,
	.author:after,
	.author:before,
	.praise-shoucang:after,
	.praise-shoucang:before,
	.news-list:after,
	ul:after,
	ul:before,
	.new-list-main-img:after,
	.new-list-main-three:after {
		content: " ";
		display: table;
		clear: both;
	}
	
	.news-list {
		width: 90%;
		margin: 0 auto;
		.new-list-main-title {
			padding: 0.5rem 0;
			font-size: 1.2rem;
		}
		.new-list-main-left-break {
			width: 100%;
		}
		.title-height {
			min-height: 3rem;
		}
		.new-list-main-img {
			padding-bottom: 0.5rem;
			img {
				display: block;
				float: left;
				width: 32%;
				height: 6rem;
				padding-right: 1%;
			}
		}
		.new-list-main-top {
			font-size: 5px;
			color: #FF0000;
			border: 1px solid #FF0000;
			border-radius: 5px;
			padding: 1px 5px;
		}
		ul {
			margin-top: 0.5rem;
			width: 300px;
			li {
				float: left;
				padding-right: 0.5rem;
				color: #CCCCCC;
			}
		}
		.new-list-main-li3 {
			padding-top: 1px;
			img {
				display: inline-block;
				width: 1rem;
				height: 0.6rem;
				opacity: 0.5;
			}
		}
		.new-list-main-three {
			.new-list-main-left {
				width: 65%;
				float: left;
				overflow: hidden;
			}
			.new-list-main-right {
				height: 6rem;
				width: 34%;
				float: right;
				overflow: hidden;
				img {
					width: 100%;
					height: 100%;
					padding: 0.5rem 0;
				}
			}
		}
	}
	/*.news-list-left,
	.news-list-right,
	{
		float: left;
	}
	
	.news-list-left {
		width: 23%;
		img {
			display: inline-block;
			height: 6rem;
			width: 6rem;
		}
		.state {
			border: 1px solid #000000;
			float: left;
			margin-right: 0.2rem;
			padding: 0.1rem 0.2rem;
			border-radius: 5px;
			line-height: 1.3em;
			color: #666666;
			border-color: #666666;
		}
		.method {
			padding: 1rem 0;
			overflow: hidden;
			text-overflow: ellipsis;
			white-space: nowrap;
		}
	}
	
	.news-list-right p:first-of-type {
		color: #888888;
		text-overflow: ellipsis;
		overflow: hidden;
		white-space: nowrap;
		padding: 1rem 0;
	}
	
	.news-list-right {
		width: 72%;
		padding-left: 5%;
		h3 {
			font-size: 1.2rem;
			color: #333333;
			text-overflow: ellipsis;
			overflow: hidden;
			white-space: nowrap;
		}
		i {
			display: inline-block;
			width: 1rem;
			height: 1.2rem;
			margin-right: 0.5rem;
			img {
				width: 100%;
			}
		}
		.span-time {
			color: #999999;
			float: left;
		}
		.span-number {
			color: #999999;
			float: right;
		}
		.author {
			text-align: right;
			padding: 0.5rem 0;
		}
		.shoucang,
		.praise {
			float: right;
			padding: 0 2rem 1rem;
		}
		.praise-shoucang {
			height: 2rem;
			margin-top: 0.5rem;
			right: 0;
			bottom: 0;
		}*/
	/*}*/
</style>