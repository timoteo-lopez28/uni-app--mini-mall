<template>
	<view class="cart-page">
		<!-- Custom Navigation Bar -->
		<view class="nav-bar" :style="{ paddingTop: statusBarHeight + 'px', height: navBarTotalHeight + 'px' }">
			<view class="nav-content">
				<text class="nav-title">购物车</text>
			</view>
		</view>
		<view class="nav-placeholder" :style="{ height: navBarTotalHeight + 'px' }"></view>

		<!-- Sub Header -->
		<view class="sub-header">
			<view class="free-ship-tag">全场包邮</view>
			<view class="manage-btn" @click="toggleManage">
				<u-icon name="setting" :size="32" color="#666666"></u-icon>
				<text class="manage-text">管理</text>
			</view>
		</view>

		<!-- Cart Content -->
		<scroll-view scroll-y class="scroll-wrap">
			<!-- Empty State -->
			<view v-if="!shippingCarInfo || shippingCarInfo.number === 0" class="empty-cart">
				<text class="empty-msg">『购物车中暂无商品』</text>
				<view class="go-shop-btn" @click="goShop">去逛逛</view>
			</view>

			<!-- Cart Items -->
			<view v-if="shippingCarInfo && shippingCarInfo.number > 0" class="cart-list">
				<u-swipe-action
					v-for="(item, index) in shippingCarInfo.items"
					:key="index"
					:show="item.show"
					:index="index"
					@click="swipeClick"
					@open="swipeOpen"
					:options="swipeOptions"
				>
					<view class="cart-item">
						<view class="item-check" @click.stop="toggleSelect(index)">
							<view class="check-circle" :class="{ checked: item.selected }">
								<u-icon v-if="item.selected" name="checkmark" :size="24" color="#FFFFFF"></u-icon>
							</view>
						</view>
						<image :src="item.pic" mode="aspectFill" class="item-img"></image>
						<view class="item-detail">
							<view class="item-name u-line-2">{{ item.name }}</view>
							<view class="item-sku">
								<text
									v-for="(sku, si) in item.sku"
									:key="'s' + si"
								>{{ sku.optionName }}:{{ sku.optionValueName }} </text>
							</view>
							<view class="item-bottom">
								<text class="item-price">¥{{ item.price }}</text>
								<u-number-box
									class="number-box"
									v-model="item.number"
									:index="index"
									:min="item.minBuyNumber"
									:max="item.stores"
									@change="numberChange"
								></u-number-box>
							</view>
						</view>
					</view>
				</u-swipe-action>
			</view>

			<view class="bottom-gap"></view>
		</scroll-view>

		<!-- Fixed Bottom Bar -->
		<view class="checkout-bar safe-area-inset-bottom">
			<view class="select-all" @click="toggleSelectAll">
				<view class="check-circle" :class="{ checked: isAllSelected }">
					<u-icon v-if="isAllSelected" name="checkmark" :size="24" color="#FFFFFF"></u-icon>
				</view>
				<text class="select-all-text">全选</text>
			</view>
			<view class="total-area">
				<text class="total-label">合计：</text>
				<text class="total-price">¥{{ totalPrice }}</text>
			</view>
			<view
				class="checkout-btn"
				:class="{ disabled: selectedCount === 0 }"
				@click="checkout"
			>
				去结算（{{ selectedCount }}）
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				statusBarHeight: 0,
				navBarTotalHeight: 88,
				shippingCarInfo: null,
				swipeOptions: [
					{
						text: '删除',
						style: { backgroundColor: '#E02020' }
					}
				],
				isManaging: false,
			}
		},
		computed: {
			selectedItems() {
				if (!this.shippingCarInfo || !this.shippingCarInfo.items) return []
				return this.shippingCarInfo.items.filter(item => item.selected)
			},
			selectedCount() {
				return this.selectedItems.length
			},
			isAllSelected() {
				if (!this.shippingCarInfo || !this.shippingCarInfo.items || this.shippingCarInfo.items.length === 0) return false
				return this.shippingCarInfo.items.every(item => item.selected)
			},
			totalPrice() {
				const total = this.selectedItems.reduce((sum, item) => {
					return sum + item.price * item.number
				}, 0)
				return total.toFixed(2)
			},
		},
		onLoad() {
			const info = uni.getSystemInfoSync()
			this.statusBarHeight = info.statusBarHeight || 0
			this.navBarTotalHeight = this.statusBarHeight + 44
		},
		onShow() {
			this.loadCart()
		},
		methods: {
			async loadCart() {
				const res = await this.$api.shippingCarInfo(this.token)
				if (res.code === 0) {
					res.data.items.forEach(item => {
						item.show = false
						item.selected = false
					})
					this.shippingCarInfo = res.data
				} else {
					this.shippingCarInfo = { number: 0, items: [], price: 0 }
				}
			},
			async numberChange(e) {
				const item = this.shippingCarInfo.items[e.index]
				const res = await this.$api.shippingCarInfoModifyNumber(this.token, item.key, e.value)
				if (res.code !== 0) {
					uni.showToast({ title: res.msg, icon: 'none' })
				} else {
					this.loadCart()
				}
			},
			swipeOpen(index) {
				this.shippingCarInfo.items.forEach(item => { item.show = false })
				this.shippingCarInfo.items[index].show = true
			},
			async swipeClick(index1, index2) {
				if (index2 === 0) {
					const item = this.shippingCarInfo.items[index1]
					await this.$api.shippingCarInfoRemoveItem(this.token, item.key)
					this.loadCart()
				}
			},
			toggleSelect(index) {
				const item = this.shippingCarInfo.items[index]
				item.selected = !item.selected
				this.$forceUpdate()
			},
			toggleSelectAll() {
				if (!this.shippingCarInfo || !this.shippingCarInfo.items) return
				const target = !this.isAllSelected
				this.shippingCarInfo.items.forEach(item => { item.selected = target })
				this.$forceUpdate()
			},
			toggleManage() {
				this.isManaging = !this.isManaging
			},
			goShop() {
				uni.switchTab({ url: '/pages/shop/index' })
			},
			checkout() {
				if (this.selectedCount === 0) {
					uni.showToast({ title: '请选择商品', icon: 'none' })
					return
				}
				uni.navigateTo({ url: '../to-pay-order/index?mod=cart' })
			},
		}
	}
</script>

<style scoped lang="scss">
	page {
		background-color: #FEF8EE;
	}

	.cart-page {
		min-height: 100vh;
		background-color: #FEF8EE;
		padding-bottom: 120rpx;
	}

	.nav-bar {
		position: fixed;
		top: 0;
		left: 0;
		right: 0;
		z-index: 100;
		background-color: #FEF8EE;
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
			font-size: 40rpx;
			font-weight: bold;
			color: #3D2B1F;
		}
	}

	/* Sub Header */
	.sub-header {
		display: flex;
		align-items: center;
		justify-content: space-between;
		padding: 16rpx 28rpx;
		background-color: #FEF8EE;

		.free-ship-tag {
			background-color: #C8832A;
			color: #FFFFFF;
			font-size: 24rpx;
			padding: 8rpx 24rpx;
			border-radius: 24rpx;
		}

		.manage-btn {
			display: flex;
			align-items: center;
			gap: 6rpx;

			.manage-text {
				font-size: 28rpx;
				color: #666666;
			}
		}
	}

	/* Scroll */
	.scroll-wrap {
		height: calc(100vh - var(--window-bottom, 0px) - 120rpx);
	}

	/* Empty State */
	.empty-cart {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		padding: 120rpx 0;

		.empty-msg {
			font-size: 28rpx;
			color: #999999;
			margin-bottom: 40rpx;
		}

		.go-shop-btn {
			background-color: #C8832A;
			color: #FFFFFF;
			font-size: 28rpx;
			padding: 16rpx 60rpx;
			border-radius: 40rpx;
		}
	}

	/* Cart List */
	.cart-list {
		padding: 8rpx 0;
	}

	.cart-item {
		display: flex;
		align-items: center;
		background-color: #FFFFFF;
		margin: 8rpx 20rpx;
		border-radius: 16rpx;
		padding: 20rpx 16rpx;
		box-shadow: 0 2rpx 8rpx rgba(0,0,0,0.04);

		.item-check {
			padding-right: 16rpx;
			flex-shrink: 0;
		}

		.check-circle {
			width: 44rpx;
			height: 44rpx;
			border-radius: 50%;
			border: 2rpx solid #CCCCCC;
			display: flex;
			align-items: center;
			justify-content: center;

			&.checked {
				background-color: #C8832A;
				border-color: #C8832A;
			}
		}

		.item-img {
			width: 180rpx;
			height: 180rpx;
			border-radius: 10rpx;
			flex-shrink: 0;
			margin-right: 16rpx;
		}

		.item-detail {
			flex: 1;
			display: flex;
			flex-direction: column;
			justify-content: space-between;
			min-height: 160rpx;

			.item-name {
				font-size: 28rpx;
				color: #333333;
				line-height: 1.4;
				margin-bottom: 8rpx;
			}

			.item-sku {
				font-size: 22rpx;
				color: #999999;
				margin-bottom: 8rpx;
			}

			.item-bottom {
				display: flex;
				align-items: center;
				justify-content: space-between;

				.item-price {
					font-size: 32rpx;
					color: #E02020;
					font-weight: bold;
				}

				.number-box {
					flex-shrink: 0;
				}
			}
		}
	}

	.bottom-gap {
		height: 40rpx;
	}

	/* Checkout Bar */
	.checkout-bar {
		position: fixed;
		bottom: 0;
		left: 0;
		right: 0;
		height: 120rpx;
		background-color: #FFFFFF;
		border-top: 2rpx solid #F0E8DC;
		display: flex;
		align-items: center;
		padding: 0 24rpx;
		z-index: 99;

		.select-all {
			display: flex;
			align-items: center;
			gap: 12rpx;
			margin-right: 20rpx;

			.check-circle {
				width: 44rpx;
				height: 44rpx;
				border-radius: 50%;
				border: 2rpx solid #CCCCCC;
				display: flex;
				align-items: center;
				justify-content: center;

				&.checked {
					background-color: #C8832A;
					border-color: #C8832A;
				}
			}

			.select-all-text {
				font-size: 28rpx;
				color: #333333;
			}
		}

		.total-area {
			flex: 1;
			display: flex;
			align-items: baseline;

			.total-label {
				font-size: 26rpx;
				color: #333333;
			}

			.total-price {
				font-size: 36rpx;
				color: #E02020;
				font-weight: bold;
			}
		}

		.checkout-btn {
			background-color: #C8832A;
			color: #FFFFFF;
			font-size: 28rpx;
			font-weight: bold;
			padding: 20rpx 32rpx;
			border-radius: 40rpx;
			white-space: nowrap;

			&.disabled {
				background-color: #CCCCCC;
			}
		}
	}
</style>
