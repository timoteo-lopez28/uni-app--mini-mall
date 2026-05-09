<template>
	<view class="my-page">
		<!-- Custom Navigation Bar -->
		<view class="nav-bar" :style="{ paddingTop: statusBarHeight + 'px', height: navBarTotalHeight + 'px' }"></view>
		<view class="nav-placeholder" :style="{ height: navBarTotalHeight + 'px' }"></view>

		<scroll-view scroll-y class="scroll-wrap">
			<!-- User Info Section -->
			<view class="user-section">
				<view class="user-row">
					<!-- #ifdef MP-WEIXIN || MP-BAIDU || MP-QQ -->
					<view class="avatar-wrap">
						<open-data type="userAvatarUrl" class="wx-avatar"></open-data>
					</view>
					<!-- #endif -->
					<!-- #ifndef MP-WEIXIN || MP-BAIDU || MP-QQ -->
					<view class="avatar-wrap" @click="goLogin">
						<image
							v-if="userDetail && userDetail.base.avatarUrl"
							:src="userDetail.base.avatarUrl"
							mode="aspectFill"
							class="avatar-img"
						></image>
						<view v-else class="avatar-placeholder">
							<u-icon name="account" :size="52" color="#FFFFFF"></u-icon>
							<text class="not-login-label">未登录</text>
						</view>
					</view>
					<!-- #endif -->

					<view class="user-info" @click="goLogin">
						<!-- #ifdef MP-WEIXIN || MP-BAIDU || MP-QQ -->
						<view class="user-name">
							<open-data type="userNickName"></open-data>
							<u-icon name="arrow-right" :size="28" color="#888888"></u-icon>
						</view>
						<!-- #endif -->
						<!-- #ifndef MP-WEIXIN || MP-BAIDU || MP-QQ -->
						<view class="user-name" v-if="userDetail">
							<text class="name-text u-line-1">用户{{ userDetail.base.id }}</text>
							<u-icon name="arrow-right" :size="28" color="#888888"></u-icon>
						</view>
						<view class="user-name" v-else>
							<text class="name-text u-line-1">点击登录</text>
							<u-icon name="arrow-right" :size="28" color="#888888"></u-icon>
						</view>
						<!-- #endif -->
						<view class="user-sub">
							<text v-if="!userDetail">点击编辑</text>
							<text v-else>用户编号：{{ userDetail.base.id }}</text>
						</view>
					</view>
				</view>
			</view>

			<!-- Design & Coupon Cards -->
			<view class="cards-row">
				<view class="feature-card design-card" @click="goDesign">
					<view class="card-header">
						<text class="card-title">我的设计</text>
						<u-icon name="arrow-right" :size="24" color="#C8832A"></u-icon>
					</view>
					<view class="card-icon-area">
						<text class="card-decor-icon">🪨</text>
						<text class="card-decor-icon sm">🏺</text>
					</view>
				</view>
				<view class="feature-card coupon-card" @click="showCoupons">
					<view class="card-header">
						<text class="card-title">我的优惠券</text>
						<u-icon name="arrow-right" :size="24" color="#6A7D5C"></u-icon>
					</view>
					<view class="card-icon-area">
						<text class="card-decor-icon">🎟</text>
					</view>
				</view>
			</view>

			<!-- My Orders -->
			<view class="orders-section">
				<view class="section-row">
					<text class="section-label">我的订单</text>
					<view class="all-orders" @click="goOrders(null)">
						<text>所有订单</text>
						<u-icon name="arrow-right" :size="24" color="#888888"></u-icon>
					</view>
				</view>
				<view class="order-icons-row">
					<view class="order-icon-item" @click="goOrders(0)">
						<u-icon name="red-packet" :size="52" color="#C8832A"></u-icon>
						<text class="order-label">待支付</text>
					</view>
					<view class="order-icon-item" @click="goOrders(1)">
						<u-icon name="gift-fill" :size="52" color="#C8832A"></u-icon>
						<text class="order-label">待发货</text>
					</view>
					<view class="order-icon-item" @click="goOrders(2)">
						<u-icon name="car" :size="52" color="#C8832A"></u-icon>
						<text class="order-label">运输中</text>
					</view>
					<view class="order-icon-item" @click="goOrders(3)">
						<u-icon name="reload" :size="52" color="#C8832A"></u-icon>
						<text class="order-label">退款/售后</text>
					</view>
				</view>
			</view>

			<!-- Settings Menu -->
			<view class="menu-section">
				<view class="menu-item" @click="goAddress">
					<view class="menu-left">
						<u-icon name="map" :size="40" color="#C8832A"></u-icon>
						<text class="menu-text">地址管理</text>
					</view>
					<u-icon name="arrow-right" :size="28" color="#CCCCCC"></u-icon>
				</view>
				<view class="menu-divider"></view>
				<view class="menu-item">
					<!-- #ifdef MP-WEIXIN || MP-BAIDU || MP-QQ -->
					<view class="menu-left" style="flex:1; position:relative;">
						<u-icon name="server-man" :size="40" color="#C8832A"></u-icon>
						<text class="menu-text">联系客服</text>
						<button open-type="contact" class="contact-btn"></button>
					</view>
					<!-- #endif -->
					<!-- #ifndef MP-WEIXIN || MP-BAIDU || MP-QQ -->
					<view class="menu-left">
						<u-icon name="server-man" :size="40" color="#C8832A"></u-icon>
						<text class="menu-text">联系客服</text>
					</view>
					<!-- #endif -->
					<u-icon name="arrow-right" :size="28" color="#CCCCCC"></u-icon>
				</view>
				<view class="menu-divider"></view>
				<view class="menu-item" @click="goAbout">
					<view class="menu-left">
						<u-icon name="question-circle" :size="40" color="#C8832A"></u-icon>
						<text class="menu-text">意见反馈</text>
					</view>
					<u-icon name="arrow-right" :size="28" color="#CCCCCC"></u-icon>
				</view>
			</view>

			<view class="version-text">version {{ version }}</view>
			<view class="bottom-gap"></view>
		</scroll-view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				statusBarHeight: 0,
				navBarTotalHeight: 44,
				userDetail: null,
				version: getApp().globalData.version,
			}
		},
		onLoad() {
			const info = uni.getSystemInfoSync()
			this.statusBarHeight = info.statusBarHeight || 0
			this.navBarTotalHeight = this.statusBarHeight + 44
		},
		onShow() {
			getApp().checkHasLoginedH5().then(isLogin => {
				if (isLogin) {
					this.loadUserDetail()
				} else {
					this.userDetail = null
				}
			})
		},
		methods: {
			async loadUserDetail() {
				const res = await this.$api.userDetail(this.token)
				if (res.code === 0) {
					if (!res.data.base.avatarUrl) {
						res.data.base.avatarUrl = '/static/images/empty.jpg'
					}
					this.userDetail = res.data
				}
			},
			goLogin() {
				if (!this.userDetail) {
					uni.navigateTo({ url: '/pages/login/login' })
				}
			},
			goDesign() {
				uni.switchTab({ url: '/pages/design/index' })
			},
			showCoupons() {
				uni.showToast({ title: '暂无优惠券', icon: 'none' })
			},
			goOrders(status) {
				const url = status !== null ? `/pages/order/index?status=${status}` : '/pages/order/index'
				uni.navigateTo({ url })
			},
			goAddress() {
				uni.navigateTo({ url: '/pages/address/index' })
			},
			goAbout() {
				uni.navigateTo({ url: '/pages/about/about?key=aboutus' })
			},
		}
	}
</script>

<style lang="scss" scoped>
	page {
		background-color: #FEF8EE;
	}

	.my-page {
		min-height: 100vh;
		background-color: #FEF8EE;
	}

	.nav-bar {
		position: fixed;
		top: 0;
		left: 0;
		right: 0;
		z-index: 100;
		background-color: #FEF8EE;
	}

	.scroll-wrap {
		height: calc(100vh - var(--window-bottom, 0px));
	}

	/* User Section */
	.user-section {
		background-color: #FEF8EE;
		padding: 20rpx 28rpx 28rpx;

		.user-row {
			display: flex;
			align-items: center;
			gap: 24rpx;
		}

		.avatar-wrap {
			flex-shrink: 0;
			width: 120rpx;
			height: 120rpx;
			border-radius: 50%;
			overflow: hidden;

			.wx-avatar {
				width: 120rpx;
				height: 120rpx;
				border-radius: 50%;
			}

			.avatar-img {
				width: 120rpx;
				height: 120rpx;
				border-radius: 50%;
			}

			.avatar-placeholder {
				width: 120rpx;
				height: 120rpx;
				border-radius: 50%;
				background-color: #AAAAAA;
				display: flex;
				flex-direction: column;
				align-items: center;
				justify-content: center;

				.not-login-label {
					font-size: 20rpx;
					color: #FFFFFF;
					margin-top: 4rpx;
				}
			}
		}

		.user-info {
			flex: 1;
			min-width: 0;

			.user-name {
				display: flex;
				align-items: center;
				gap: 8rpx;
				margin-bottom: 8rpx;

				.name-text {
					font-size: 30rpx;
					color: #333333;
					font-weight: 500;
					flex: 1;
				}
			}

			.user-sub {
				font-size: 24rpx;
				color: #999999;
			}
		}
	}

	/* Cards Row */
	.cards-row {
		display: flex;
		gap: 16rpx;
		padding: 0 20rpx 20rpx;
	}

	.feature-card {
		flex: 1;
		border-radius: 20rpx;
		padding: 24rpx 20rpx;
		min-height: 180rpx;
		display: flex;
		flex-direction: column;
		justify-content: space-between;

		.card-header {
			display: flex;
			align-items: center;
			justify-content: space-between;

			.card-title {
				font-size: 28rpx;
				font-weight: bold;
			}
		}

		.card-icon-area {
			display: flex;
			align-items: flex-end;
			justify-content: flex-end;
			margin-top: 16rpx;

			.card-decor-icon {
				font-size: 60rpx;

				&.sm {
					font-size: 40rpx;
					margin-left: 8rpx;
				}
			}
		}
	}

	.design-card {
		background: linear-gradient(135deg, #FFF3DC 0%, #FFE4A8 100%);
		border: 2rpx solid #F5D29A;

		.card-title {
			color: #C8832A;
		}
	}

	.coupon-card {
		background: linear-gradient(135deg, #EFF5EA 0%, #D8EBD0 100%);
		border: 2rpx solid #B8D8A8;

		.card-title {
			color: #4A7A3A;
		}
	}

	/* Orders Section */
	.orders-section {
		background-color: #FFFFFF;
		border-radius: 20rpx;
		margin: 0 20rpx 20rpx;
		padding: 24rpx;
		box-shadow: 0 2rpx 12rpx rgba(0,0,0,0.04);

		.section-row {
			display: flex;
			align-items: center;
			justify-content: space-between;
			margin-bottom: 24rpx;

			.section-label {
				font-size: 30rpx;
				font-weight: bold;
				color: #333333;
			}

			.all-orders {
				display: flex;
				align-items: center;
				font-size: 24rpx;
				color: #888888;
				gap: 4rpx;
			}
		}

		.order-icons-row {
			display: flex;
			justify-content: space-around;

			.order-icon-item {
				display: flex;
				flex-direction: column;
				align-items: center;
				gap: 12rpx;

				.order-label {
					font-size: 22rpx;
					color: #555555;
					white-space: nowrap;
				}
			}
		}
	}

	/* Settings Menu */
	.menu-section {
		background-color: #FFFFFF;
		border-radius: 20rpx;
		margin: 0 20rpx 20rpx;
		padding: 0 24rpx;
		box-shadow: 0 2rpx 12rpx rgba(0,0,0,0.04);

		.menu-item {
			display: flex;
			align-items: center;
			justify-content: space-between;
			padding: 28rpx 0;
			position: relative;

			.menu-left {
				display: flex;
				align-items: center;
				gap: 16rpx;
				flex: 1;

				.menu-text {
					font-size: 28rpx;
					color: #333333;
				}

				.contact-btn {
					position: absolute;
					top: 0;
					left: 0;
					width: 100%;
					height: 100%;
					opacity: 0;
					z-index: 10;
				}
			}
		}

		.menu-divider {
			height: 2rpx;
			background-color: #F5EFE6;
			margin-left: 56rpx;
		}
	}

	.version-text {
		text-align: center;
		font-size: 22rpx;
		color: #CCCCCC;
		padding: 20rpx 0;
	}

	.bottom-gap {
		height: 40rpx;
	}
</style>
