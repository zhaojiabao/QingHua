<template>
	<!--该页面为banner详情页面-->
	<div class="banner-detil">
		<iframe :src="URL+'?ticket='+$store.state.ticket" width="100%" height="100%" frameborder="no"></iframe>
	</div>
</template>

<script>
	import { indexBanner, userAction } from "../../api/api"
	export default {
		data() {
			return {
				URL: ''
			}
		},
		mounted() {
			this.$store.state.header.title = "资讯详情";
			//实现banner切换
			indexBanner().then(rs => {
				rs.data.forEach((item, index) => {
					if(item.cms_id == this.$route.params.ban_id) {
						this.URL = item.url;
					}
					for(var i=0;i<item.length;i++){
						this.$wechat.ready(() => {
						var title = item[i].cms_title;
						var link = window.location.href;
						var imgUrl = item[i].cms_pic;
						this.$wechat.onMenuShareTimeline({
							title: title,
							link: link,
							imgUrl: imgUrl
						});
						this.$wechat.onMenuShareAppMessage({
							title: title,
							desc: item[i].cms_title,
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
					}
				})

			});
			//用户行为
			userAction({
				ticket: this.$store.state.ticket,
				module: 19,
				cur_version: this.$store.state.cur_version
			})
		}
	}
</script>

<style>
	.banner-detil {
		width: 100vw;
		height: 95vh;
	}
</style>