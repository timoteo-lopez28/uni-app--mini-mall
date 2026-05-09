<template>
	<view class="shop-page">
		<!-- Custom Navigation Bar -->
		<view class="nav-bar" :style="{ paddingTop: statusBarHeight + 'px', height: navBarTotalHeight + 'px' }">
			<view class="nav-content">
				<text class="nav-title">手工艺品商城</text>
			</view>
		</view>
		<view class="nav-placeholder" :style="{ height: navBarTotalHeight + 'px' }"></view>

		<scroll-view scroll-y class="scroll-wrap" @scrolltolower="loadMore">
			<!-- Section Header -->
			<view class="section-header">
				<view class="sh-title">定制</view>
				<view class="sh-subtitle">购买手工艺品，享受免费定制刻字服务</view>
			</view>

			<!-- Category Tabs -->
			<scroll-view scroll-x class="category-scroll" :show-scrollbar="false">
				<view class="category-list">
					<view
						v-for="(cat, index) in categories"
						:key="index"
						class="category-tab"
						:class="{ active: currentCategory === index }"
						@click="switchCategory(index)"
					>
						{{ cat.name }}
					</view>
				</view>
			</scroll-view>

			<!-- Product Grid -->
			<view class="product-grid">
				<view v-if="!goodsList || goodsList.length === 0" class="empty-state">
					<text class="empty-icon">🏺</text>
					<text class="empty-text">暂无商品，稍后再来~</text>
				</view>
				<view
					v-for="(item, index) in goodsList"
					:key="'p' + index"
					class="product-card"
					@click="goGoods(item.id)"
				>
					<image :src="item.pic" mode="aspectFill" class="product-img"></image>
					<view class="product-info">
						<view class="product-name u-line-2">{{ item.name }}</view>
						<view class="product-price-row">
							<text class="product-price">¥{{ item.minPrice }}</text>
						</view>
						<view class="product-desc u-line-1">{{ item.intro || item.name }}</view>
					</view>
				</view>
			</view>

			<view v-if="loading" class="loading-tip">加载中...</view>
			<view v-if="noMore" class="loading-tip">已加载全部商品</view>
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
				categories: [{ id: 0, name: '全部' }],
				currentCategory: 0,
				loading: false,
				noMore: false,
				page: 1,
				pageSize: 10,
			}
		},
		onLoad() {
			const info = uni.getSystemInfoSync()
			this.statusBarHeight = info.statusBarHeight || 0
			this.navBarTotalHeight = this.statusBarHeight + 44
			this.loadCategories()
		},
		onShow() {
			if (this.goodsList.length === 0) {
				this.fetchGoods()
			}
		},
		methods: {
			async loadCategories() {
				const res = await this.$api.goodsCategory()
				if (res.code === 0 && res.data) {
					this.categories = [{ id: 0, name: '全部' }, ...res.data]
				}
				this.fetchGoods()
			},
			async fetchGoods(reset) {
				if (this.loading) return
				if (reset) {
					this.page = 1
					this.goodsList = []
					this.noMore = false
				}
				this.loading = true
				const cat = this.categories[this.currentCategory]
				const res = await this.$api.goodsv2({
					token: this.token,
					categoryId: cat && cat.id ? cat.id : '',
					page: this.page,
					limit: this.pageSize,
				})
				this.loading = false
				if (res.code === 0 && res.data && res.data.result) {
					this.goodsList = [...this.goodsList, ...res.data.result]
					if (res.data.result.length < this.pageSize) {
						this.noMore = true
					}
				} else {
					this.noMore = true
				}
			},
			switchCategory(index) {
				if (this.currentCategory === index) return
				this.currentCategory = index
				this.fetchGoods(true)
			},
			loadMore() {
				if (this.noMore || this.loading) return
				this.page++
				this.fetchGoods()
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

	.shop-page {
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

	/* Section Header */
	.section-header {
		background-color: #FFFFFF;
		padding: 24rpx 28rpx 20rpx;
		border-bottom: 2rpx solid #F0E8DC;

		.sh-title {
			font-size: 36rpx;
			font-weight: bold;
			color: #3D2B1F;
			margin-bottom: 8rpx;
		}

		.sh-subtitle {
			font-size: 24rpx;
			color: #888888;
		}
	}

	/* Category Tabs */
	.category-scroll {
		background-color: #FFFFFF;
		white-space: nowrap;
		margin-bottom: 16rpx;
	}

	.category-list {
		display: inline-flex;
		padding: 16rpx 18rpx;
		gap: 16rpx;
	}

	.category-tab {
		display: inline-block;
		padding: 10rpx 28rpx;
		border-radius: 32rpx;
		font-size: 26rpx;
		color: #666666;
		background-color: #F5EFE6;
		border: 2rpx solid transparent;
		white-space: nowrap;

		&.active {
			background-color: #FFF4E6;
			color: #C8832A;
			border-color: #C8832A;
			font-weight: bold;
		}
	}

	/* Product Grid */
	.product-grid {
		display: flex;
		flex-wrap: wrap;
		padding: 0 18rpx;
		gap: 16rpx;
	}

	.empty-state {
		width: 100%;
		padding: 80rpx 0;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;

		.empty-icon {
			font-size: 80rpx;
			margin-bottom: 20rpx;
		}

		.empty-text {
			font-size: 28rpx;
			color: #999999;
		}
	}

	.product-card {
		width: calc(50% - 8rpx);
		background-color: #FFFFFF;
		border-radius: 16rpx;
		overflow: hidden;
		box-shadow: 0 2rpx 12rpx rgba(0,0,0,0.06);

		.product-img {
			width: 100%;
			height: 340rpx;
		}

		.product-info {
			padding: 16rpx;
		}

		.product-name {
			font-size: 28rpx;
			color: #333333;
			line-height: 1.4;
			margin-bottom: 10rpx;
			font-weight: 500;
		}

		.product-price-row {
			margin-bottom: 8rpx;
		}

		.product-price {
			font-size: 32rpx;
			color: #E02020;
			font-weight: bold;
		}

		.product-desc {
			font-size: 22rpx;
			color: #999999;
			line-height: 1.4;
		}
	}

	.loading-tip {
		text-align: center;
		font-size: 24rpx;
		color: #999999;
		padding: 20rpx 0;
	}

	.bottom-gap {
		height: 40rpx;
	}
</style>
