<template>
	<view class="home-page">
		<!-- Custom Navigation Bar -->
		<view class="nav-bar" :style="{ paddingTop: statusBarHeight + 'px', height: navBarHeight + 'px' }">
			<view class="nav-content">
				<text class="nav-title">手工艺品商城</text>
			</view>
		</view>
		<view class="nav-placeholder" :style="{ height: (statusBarHeight + navBarHeight) + 'px' }"></view>

		<!-- Scrollable Content -->
		<scroll-view scroll-y class="scroll-wrap">
			<!-- Banner Swiper -->
			<swiper
				class="banner-swiper"
				circular
				autoplay
				:interval="3500"
				indicator-dots
				indicator-active-color="#C8832A"
				indicator-color="rgba(255,255,255,0.5)"
			>
				<swiper-item
					v-for="(item, index) in bannerGoods"
					:key="'b' + index"
					@click="goGoods(item.id)"
				>
					<view class="banner-item">
						<image :src="item.pic" mode="aspectFill" class="banner-img"></image>
						<view class="banner-mask">
							<view class="banner-tag">手工精品</view>
							<view class="banner-name u-line-2">{{ item.name }}</view>
							<view class="banner-action">去选购 &gt;&gt;&gt;</view>
						</view>
					</view>
				</swiper-item>
				<swiper-item v-if="!bannerGoods || bannerGoods.length === 0">
					<view class="banner-placeholder">
						<view class="ph-inner">
							<view class="ph-tag">🏺 手工艺品</view>
							<view class="ph-title">匠心精品  特卖盛宴</view>
							<view class="ph-action">立即选购 &gt;&gt;&gt;</view>
						</view>
					</view>
				</swiper-item>
			</swiper>

			<!-- Featured Section -->
			<view class="featured-section" @click="goDesign">
				<view class="featured-text-box">
					<view class="feat-title1">匠心制作</view>
					<view class="feat-title2">可见天地</view>
					<view class="feat-desc">每件手工艺品都凝聚着匠人的心血</view>
					<view class="feat-btn">立即定制</view>
				</view>
			</view>

			<!-- Recommend Section -->
			<view class="section-header">
				<text class="section-title">精品手工艺品</text>
			</view>
			<scroll-view scroll-x class="product-scroll" :show-scrollbar="false">
				<view class="product-list">
					<view
						v-for="(item, index) in goodsList"
						:key="'g' + index"
						class="product-card"
						@click="goGoods(item.id)"
					>
						<image :src="item.pic" mode="aspectFill" class="product-img"></image>
						<view class="product-name u-line-2">{{ item.name }}</view>
						<view class="product-price">¥{{ item.minPrice }}</view>
					</view>
				</view>
			</scroll-view>

			<view class="bottom-gap"></view>
		</scroll-view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				statusBarHeight: 0,
				navBarHeight: 88,
				goodsList: [],
				bannerGoods: [],
			}
		},
		onLoad() {
			const info = uni.getSystemInfoSync()
			this.statusBarHeight = info.statusBarHeight || 0
			// #ifdef MP-WEIXIN
			uni.$once('loginOK', () => {
				this.loadGoods()
			})
			// #endif
			// #ifndef MP-WEIXIN
			this.loadGoods()
			// #endif
		},
		methods: {
			async loadGoods() {
				const data = { token: this.token, recommendStatus: 1 }
				const res = await this.$api.goodsv2(data)
				if (res.code === 0 && res.data && res.data.result) {
					this.goodsList = res.data.result
					this.bannerGoods = res.data.result.slice(0, 5)
				}
			},
			goGoods(id) {
				uni.navigateTo({ url: '/pages/goods/details?id=' + id })
			},
			goDesign() {
				uni.switchTab({ url: '/pages/design/index' })
			},
		}
	}
</script>

<style scoped lang="scss">
	page {
		background-color: #F5EFE6;
	}

	.home-page {
		min-height: 100vh;
		background-color: #F5EFE6;
	}

	.nav-bar {
		position: fixed;
		top: 0;
		left: 0;
		right: 0;
		z-index: 100;
		background-color: #FFFFFF;
		display: flex;
		flex-direction: column;
		justify-content: flex-end;

		.nav-content {
			height: 88rpx;
			display: flex;
			align-items: center;
			padding: 0 30rpx;
		}

		.nav-title {
			font-size: 36rpx;
			font-weight: bold;
			color: #3D2B1F;
			letter-spacing: 2rpx;
		}
	}

	.scroll-wrap {
		height: calc(100vh - var(--window-bottom, 0px));
	}

	/* Banner */
	.banner-swiper {
		width: 100%;
		height: 420rpx;
	}

	.banner-item {
		width: 100%;
		height: 420rpx;
		position: relative;
		overflow: hidden;

		.banner-img {
			width: 100%;
			height: 100%;
		}

		.banner-mask {
			position: absolute;
			bottom: 0;
			left: 0;
			right: 0;
			padding: 24rpx 28rpx 32rpx;
			background: linear-gradient(to top, rgba(0,0,0,0.6), transparent);

			.banner-tag {
				display: inline-block;
				background-color: #C8832A;
				color: #FFFFFF;
				font-size: 22rpx;
				padding: 4rpx 16rpx;
				border-radius: 20rpx;
				margin-bottom: 10rpx;
			}

			.banner-name {
				font-size: 32rpx;
				color: #FFFFFF;
				font-weight: bold;
				line-height: 1.4;
				margin-bottom: 8rpx;
			}

			.banner-action {
				font-size: 24rpx;
				color: #F5D29A;
			}
		}
	}

	.banner-placeholder {
		width: 100%;
		height: 420rpx;
		background: linear-gradient(135deg, #8B4513 0%, #C8832A 60%, #E8A040 100%);
		display: flex;
		align-items: center;
		justify-content: center;

		.ph-inner {
			text-align: center;

			.ph-tag {
				font-size: 32rpx;
				margin-bottom: 16rpx;
			}

			.ph-title {
				font-size: 40rpx;
				color: #FFFFFF;
				font-weight: bold;
				letter-spacing: 4rpx;
				margin-bottom: 16rpx;
			}

			.ph-action {
				font-size: 26rpx;
				color: #FFE4B5;
			}
		}
	}

	/* Featured Section */
	.featured-section {
		margin: 20rpx 0;
		height: 360rpx;
		background: linear-gradient(135deg, #5C3317 0%, #8B5E3C 50%, #C8832A 100%);
		position: relative;
		overflow: hidden;
		display: flex;
		align-items: center;

		&::before {
			content: '';
			position: absolute;
			right: -60rpx;
			top: -60rpx;
			width: 280rpx;
			height: 280rpx;
			border-radius: 50%;
			background: rgba(255,255,255,0.06);
		}

		&::after {
			content: '';
			position: absolute;
			right: 40rpx;
			bottom: -80rpx;
			width: 200rpx;
			height: 200rpx;
			border-radius: 50%;
			background: rgba(255,255,255,0.04);
		}

		.featured-text-box {
			padding-left: 48rpx;
			z-index: 1;

			.feat-title1 {
				font-size: 64rpx;
				color: #FFFFFF;
				font-weight: bold;
				line-height: 1.2;
				letter-spacing: 4rpx;
			}

			.feat-title2 {
				font-size: 64rpx;
				color: #FFFFFF;
				font-weight: bold;
				line-height: 1.2;
				letter-spacing: 4rpx;
				margin-bottom: 16rpx;
			}

			.feat-desc {
				font-size: 24rpx;
				color: rgba(255,255,255,0.75);
				margin-bottom: 32rpx;
			}

			.feat-btn {
				display: inline-block;
				background-color: #F5D29A;
				color: #5C3317;
				font-size: 28rpx;
				font-weight: bold;
				padding: 16rpx 44rpx;
				border-radius: 40rpx;
			}
		}
	}

	/* Recommend Section */
	.section-header {
		padding: 24rpx 28rpx 12rpx;

		.section-title {
			font-size: 34rpx;
			font-weight: bold;
			color: #3D2B1F;
		}
	}

	.product-scroll {
		white-space: nowrap;
		padding: 0 18rpx 16rpx;
	}

	.product-list {
		display: inline-flex;
		flex-direction: row;
		padding: 4rpx 0;
	}

	.product-card {
		display: inline-block;
		width: 280rpx;
		margin: 0 10rpx;
		background-color: #FFFFFF;
		border-radius: 16rpx;
		overflow: hidden;
		box-shadow: 0 2rpx 12rpx rgba(0,0,0,0.08);
		vertical-align: top;
		white-space: normal;

		.product-img {
			width: 280rpx;
			height: 280rpx;
		}

		.product-name {
			padding: 12rpx 16rpx 4rpx;
			font-size: 26rpx;
			color: #333333;
			line-height: 1.4;
		}

		.product-price {
			padding: 4rpx 16rpx 16rpx;
			font-size: 28rpx;
			color: #C8832A;
			font-weight: bold;
		}
	}

	.bottom-gap {
		height: 40rpx;
	}
</style>
