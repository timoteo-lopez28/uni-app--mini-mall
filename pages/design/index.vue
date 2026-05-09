<template>
	<view class="design-page">
		<!-- Custom Navigation Bar -->
		<view class="nav-bar" :style="{ paddingTop: statusBarHeight + 'px', height: navBarTotalHeight + 'px' }">
			<view class="nav-content">
				<text class="nav-title">定制设计</text>
			</view>
		</view>
		<view class="nav-placeholder" :style="{ height: navBarTotalHeight + 'px' }"></view>

		<scroll-view scroll-y class="scroll-wrap">
			<!-- Hero Section -->
			<view class="hero-section">
				<view class="hero-icon">🏺</view>
				<view class="hero-title">将您的创意变为现实</view>
				<view class="hero-desc">专业匠人团队，精心制作每一件手工艺品</view>
			</view>

			<!-- Service Cards -->
			<view class="section-label">定制服务</view>
			<view class="service-grid">
				<view
					v-for="(item, index) in services"
					:key="index"
					class="service-card"
					@click="goShop"
				>
					<text class="service-icon">{{ item.icon }}</text>
					<view class="service-name">{{ item.name }}</view>
					<view class="service-desc">{{ item.desc }}</view>
				</view>
			</view>

			<!-- Process Section -->
			<view class="section-label">定制流程</view>
			<view class="process-list">
				<view v-for="(step, index) in steps" :key="index" class="process-item">
					<view class="step-number">{{ index + 1 }}</view>
					<view class="step-content">
						<view class="step-title">{{ step.title }}</view>
						<view class="step-desc">{{ step.desc }}</view>
					</view>
				</view>
			</view>

			<!-- CTA -->
			<view class="cta-section">
				<view class="cta-title">立即开始您的专属定制</view>
				<view class="cta-desc">购买手工艺品，享受免费定制刻字服务</view>
				<view class="cta-btn" @click="goShop">前往商城选购</view>
			</view>

			<!-- Recommend Products -->
			<view class="section-label">热门定制</view>
			<view class="product-grid">
				<view
					v-for="(item, index) in goodsList"
					:key="'g' + index"
					class="product-item"
					@click="goGoods(item.id)"
				>
					<image :src="item.pic" mode="aspectFill" class="product-img"></image>
					<view class="product-name u-line-2">{{ item.name }}</view>
					<view class="product-price">¥{{ item.minPrice }}</view>
				</view>
			</view>

			<view class="bottom-gap"></view>
		</scroll-view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				statusBarHeight: 0,
				navBarTotalHeight: 88,
				goodsList: [],
				services: [
					{ icon: '🪨', name: '石雕定制', desc: '青田石、寿山石等名石雕刻' },
					{ icon: '🏺', name: '陶瓷定制', desc: '景德镇手绘瓷器定制' },
					{ icon: '🎋', name: '竹编定制', desc: '传统竹编工艺品定制' },
					{ icon: '🪵', name: '木雕定制', desc: '花梨木、檀木精雕细琢' },
					{ icon: '🧵', name: '刺绣定制', desc: '苏绣、蜀绣等传统刺绣' },
					{ icon: '💍', name: '银饰定制', desc: '苗银手工打造独特饰品' },
				],
				steps: [
					{ title: '选择商品', desc: '在商城中选择您喜欢的手工艺品类型' },
					{ title: '提交需求', desc: '填写您的定制要求，如文字、图案、尺寸等' },
					{ title: '匠人制作', desc: '专业匠人根据您的需求精心制作' },
					{ title: '质检发货', desc: '严格质检后精心包装发货到您手中' },
				],
			}
		},
		onLoad() {
			const info = uni.getSystemInfoSync()
			this.statusBarHeight = info.statusBarHeight || 0
			this.navBarTotalHeight = this.statusBarHeight + 44
			this.loadGoods()
		},
		methods: {
			async loadGoods() {
				const res = await this.$api.goodsv2({ token: this.token })
				if (res.code === 0 && res.data && res.data.result) {
					this.goodsList = res.data.result.slice(0, 4)
				}
			},
			goShop() {
				uni.switchTab({ url: '/pages/shop/index' })
			},
			goGoods(id) {
				uni.navigateTo({ url: '/pages/goods/details?id=' + id })
			},
		}
	}
</script>

<style scoped lang="scss">
	page {
		background-color: #F5EFE6;
	}

	.design-page {
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
		}
	}

	.scroll-wrap {
		height: calc(100vh - var(--window-bottom, 0px));
	}

	/* Hero */
	.hero-section {
		background: linear-gradient(135deg, #5C3317 0%, #8B5E3C 60%, #C8832A 100%);
		padding: 60rpx 40rpx 50rpx;
		text-align: center;

		.hero-icon {
			font-size: 88rpx;
			margin-bottom: 20rpx;
		}

		.hero-title {
			font-size: 40rpx;
			color: #FFFFFF;
			font-weight: bold;
			margin-bottom: 16rpx;
			letter-spacing: 2rpx;
		}

		.hero-desc {
			font-size: 26rpx;
			color: rgba(255,255,255,0.8);
			line-height: 1.6;
		}
	}

	/* Section Label */
	.section-label {
		font-size: 32rpx;
		font-weight: bold;
		color: #3D2B1F;
		padding: 32rpx 28rpx 16rpx;
	}

	/* Service Grid */
	.service-grid {
		display: flex;
		flex-wrap: wrap;
		padding: 0 18rpx 8rpx;
		gap: 16rpx;
	}

	.service-card {
		width: calc(50% - 8rpx);
		background-color: #FFFFFF;
		border-radius: 16rpx;
		padding: 28rpx 20rpx;
		box-shadow: 0 2rpx 12rpx rgba(0,0,0,0.06);

		.service-icon {
			font-size: 52rpx;
			display: block;
			margin-bottom: 12rpx;
		}

		.service-name {
			font-size: 28rpx;
			font-weight: bold;
			color: #3D2B1F;
			margin-bottom: 8rpx;
		}

		.service-desc {
			font-size: 22rpx;
			color: #888888;
			line-height: 1.5;
		}
	}

	/* Process */
	.process-list {
		padding: 0 28rpx 8rpx;
	}

	.process-item {
		display: flex;
		align-items: flex-start;
		margin-bottom: 28rpx;

		.step-number {
			width: 52rpx;
			height: 52rpx;
			border-radius: 50%;
			background-color: #C8832A;
			color: #FFFFFF;
			font-size: 28rpx;
			font-weight: bold;
			display: flex;
			align-items: center;
			justify-content: center;
			flex-shrink: 0;
			margin-right: 20rpx;
			margin-top: 4rpx;
		}

		.step-content {
			flex: 1;

			.step-title {
				font-size: 28rpx;
				font-weight: bold;
				color: #3D2B1F;
				margin-bottom: 6rpx;
			}

			.step-desc {
				font-size: 24rpx;
				color: #666666;
				line-height: 1.5;
			}
		}
	}

	/* CTA */
	.cta-section {
		margin: 8rpx 28rpx 8rpx;
		background-color: #FFF8EE;
		border: 2rpx solid #E8C87A;
		border-radius: 20rpx;
		padding: 36rpx 28rpx;
		text-align: center;

		.cta-title {
			font-size: 32rpx;
			font-weight: bold;
			color: #3D2B1F;
			margin-bottom: 12rpx;
		}

		.cta-desc {
			font-size: 24rpx;
			color: #888888;
			margin-bottom: 28rpx;
		}

		.cta-btn {
			display: inline-block;
			background-color: #C8832A;
			color: #FFFFFF;
			font-size: 28rpx;
			font-weight: bold;
			padding: 18rpx 60rpx;
			border-radius: 40rpx;
		}
	}

	/* Product Grid */
	.product-grid {
		display: flex;
		flex-wrap: wrap;
		padding: 0 18rpx;
		gap: 16rpx;
	}

	.product-item {
		width: calc(50% - 8rpx);
		background-color: #FFFFFF;
		border-radius: 16rpx;
		overflow: hidden;
		box-shadow: 0 2rpx 12rpx rgba(0,0,0,0.06);

		.product-img {
			width: 100%;
			height: 330rpx;
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
