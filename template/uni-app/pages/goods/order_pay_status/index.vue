<template>
	<view :style="colorStyle">
		<view class="payment-status" v-if="(!orderLottery || !order_pay_info.paid || is_gift) && loading && lotteryLoading">
			<!--失败时： 用icon-iconfontguanbi fail替换icon-duihao2 bg-color-->
			<view class="iconfont icons icon-duihao2 bg-color" v-if="order_pay_info.paid || order_pay_info.pay_type == 'offline'"></view>
			<view class="iconfont icons icon-iconfontguanbi" v-else></view>
			<!-- 失败时：订单支付失败 -->
			<view class="status" v-if="order_pay_info.pay_type != 'offline'">
				{{ order_pay_info.paid ? $t(`订单支付成功`) : $t(payType ? `订单支付中` : `订单支付失败`) }}
			</view>
			<view class="status" v-else>{{ $t(`订单创建成功`) }}</view>
			<view class="wrapper">
				<view class="item acea-row row-between-wrapper">
					<view>{{ $t(`订单号`) }}</view>
					<view class="itemCom">{{ orderId }}</view>
				</view>
				<view class="item acea-row row-between-wrapper">
					<view>{{ $t(`下单时间`) }}</view>
					<view class="itemCom">{{ order_pay_info._add_time }}</view>
				</view>
				<view class="item acea-row row-between-wrapper">
					<view>{{ $t(`支付方式`) }}</view>
					<view class="itemCom">{{ $t(order_pay_info._status._payType) || $t(`暂未支付`) }}</view>
				</view>
				<view class="item acea-row row-between-wrapper">
					<view>{{ $t(`支付金额`) }}</view>
					<view class="itemCom">{{ order_pay_info.pay_price }}</view>
				</view>
				<!--失败时加上这个  -->
				<view class="item acea-row row-between-wrapper" v-if="order_pay_info.paid == 0 && order_pay_info.pay_type != 'offline'">
					<view>{{ $t(`失败原因`) }}</view>
					<view class="itemCom">{{ $t(`未支付`) }}</view>
				</view>
			</view>
			<view v-if="order_pay_info.paid != 0 && is_gift !== 0" @click="giftModalShow = true">
				<button class="returnBnt bg-color" hover-class="none">{{ $t(`送给好友`) }}</button>
			</view>
			<!--失败时： 重新购买 -->
			<view @tap="goOrderDetails" v-if="status == 0">
				<button formType="submit" class="returnBnt bg-color" hover-class="none">{{ $t(`查看订单`) }}</button>
			</view>
			<!-- #ifdef H5 -->
			<view @tap="getOrderPayInfo" v-if="order_pay_info.paid == 0">
				<button class="returnBnt bg-color" hover-class="none">{{ $t(`刷新支付状态`) }}</button>
			</view>
			<!-- #endif -->
			<view @tap="goOrderDetails" v-if="order_pay_info.paid == 0 && status == 1">
				<button class="returnBnt bg-color" hover-class="none">{{ $t(`重新购买`) }}</button>
			</view>
			<view @tap="goOrderDetails" v-if="order_pay_info.paid == 0 && status == 2">
				<button class="returnBnt bg-color" hover-class="none">{{ $t(`重新支付`) }}</button>
			</view>
			<button
				@click="goPink(order_pay_info.pink_id)"
				class="returnBnt cart-color"
				formType="submit"
				hover-class="none"
				v-if="order_pay_info.pink_id && order_pay_info.paid != 0 && status != 2 && status != 1"
			>
				{{ $t(`邀请好友参团`) }}
			</button>
			<button @click="goIndex" class="returnBnt cart-color" formType="submit" hover-class="none" v-else>{{ $t(`返回首页`) }}</button>
			<view class="coupons" v-if="couponList.length">
				<view class="title acea-row row-center-wrapper">
					<view class="line"></view>
					<view class="name">{{ $t(`赠送优惠券`) }}</view>
					<view class="line"></view>
				</view>
				<view class="list">
					<view class="item acea-row row-between-wrapper" v-for="(item, index) in couponList" :key="index" v-if="index < 2 || !couponsHidden">
						<view class="moneyCon acea-row row-between-wrapper">
							<view class="price acea-row row-center-wrapper">
								<view>
									{{ $t(`￥`) }}
									<text>{{ item.coupon_price }}</text>
								</view>
							</view>
						</view>
						<view class="text">
							<view class="name line1">{{ item.coupon_title }}</view>
							<view class="priceMin">{{ $t(`满`) }}{{ item.use_min_price }}{{ $t(`元可用`) }}</view>
							<view class="time">{{ $t(`有效期`) }}:{{ item.add_time ? item.add_time + '-' : '' }}{{ item.end_time }}</view>
						</view>
					</view>
					<view class="open acea-row row-center-wrapper" @click="openTap" v-if="couponList.length > 2">
						{{ couponsHidden ? $t(`更多`) : $t(`关闭`) }}
						<text class="iconfont" :class="couponsHidden == true ? 'icon-xiangxia' : 'icon-xiangshang'"></text>
					</view>
				</view>
			</view>
		</view>
		<lotteryModel
			v-show="orderLottery && order_pay_info.paid && loading && lotteryLoading && !is_gift"
			:options="options"
			@orderDetails="goOrderDetails"
			@lotteryShow="getOrderLottery"
		></lotteryModel>
		<giftModal :aleartStatus="giftModalShow" :giftData="giftData" @shareH5="shareH5" @close="giftModalShow = false"></giftModal>
		<view class="mask" v-if="giftModalShow"></view>
		<canvas class="canvas" canvas-id="posterCanvas"></canvas>
		<view class="share-box" v-if="H5ShareBox">
			<image :src="imgHost + '/statics/images/share-info.png'" @click="H5ShareBox = false"></image>
		</view>
	</view>
</template>

<script>
import { userShare } from '@/api/user.js';
import lotteryModel from './payLottery.vue';
import giftModal from './components/giftModal.vue';
import { getOrderDetail, orderCoupon } from '@/api/order.js';
import { openOrderSubscribe } from '@/utils/SubscribeMessage.js';
import { toLogin } from '@/libs/login.js';
import { mapGetters } from 'vuex';
// #ifdef MP
import authorize from '@/components/Authorize';
// #endif
import colors from '@/mixins/color';
import { HTTP_REQUEST_URL } from '@/config/app';
export default {
	components: {
		lotteryModel,
		giftModal,
		// #ifdef MP
		authorize
		// #endif
	},
	mixins: [colors],
	data() {
		return {
			imgHost: HTTP_REQUEST_URL,
			loading: false,
			lotteryLoading: false,
			orderLottery: false,
			orderId: '',
			order_pay_info: {
				paid: 1,
				_status: {}
			},
			isAuto: false, //没有授权的不会自动授权
			isShowAuth: false, //是否隐藏授权
			status: 0,
			msg: '',
			couponsHidden: true,
			couponList: [],
			options: null,
			payType: '',
			giftData: {},
			giftModalShow: false,
			H5ShareBox: false,
			is_gift: 0,
			storeInfo: {},
			mpGiftImg: HTTP_REQUEST_URL + '/statics/images/gift_share.jpg'
		};
	},
	computed: mapGetters(['isLogin']),
	watch: {
		isLogin: {
			handler: function (newV, oldV) {
				if (newV) {
					this.getOrderPayInfo();
				}
			},
			deep: true
		}
	},
	onLoad(options) {
		this.options = options;
		if (!options.order_id)
			return this.$util.Tips(
				{
					title: this.$t(`缺少参数无法查看订单支付状态`)
				},
				{
					tab: 3,
					url: 1
				}
			);
		this.orderId = options.order_id;
		this.status = options.status || 0;
		this.msg = options.msg || '';
		this.payType = options.payType || '';

		// // #ifdef H5
		// document.addEventListener('visibilitychange', (e) => {
		// 	let state = document.visibilityState
		// 	if (state == 'hidden') {
		// 		console.log('用户离开了');
		// 	}
		// 	if (state == 'visible') {
		// 		this.getOrderPayInfo();
		// 	}
		// });
		// // #endif
	},
	onShow() {
		if (this.isLogin) {
			this.getOrderPayInfo();
		} else {
			toLogin();
		}
	},
	/**
	 * 用户点击右上角分享
	 */
	// #ifdef MP
	onShareAppMessage: function () {
		let that = this;
		userShare();
		return {
			title: that.giftData.message || '',
			imageUrl: that.mpGiftImg || '',
			path: '/pages/goods/receive_gift/index?id=' + this.giftData.id + '&spid=' + this.$store.state.app.uid
		};
	},
	onShareTimeline() {
		let that = this;
		userShare();
		return {
			title: that.giftData.message,
			query: {
				id: that.id,
				spid: that.uid || 0
			},
			imageUrl: that.mpGiftImg
		};
	},
	// #endif
	methods: {
		// #ifdef H5
		setOpenShare() {
			let that = this;
			if (that.$wechat.isWeixin()) {
				let configAppMessage = {
					desc: this.giftData.message,
					title: this.giftData.title,
					link: window.location.protocol + '//' + window.location.host + '/pages/goods/receive_gift/index?id=' + this.giftData.id + '&spid=' + that.$store.state.app.uid,
					imgUrl: that.mpGiftImg
				};
				that.$wechat
					.wechatEvevt(['updateAppMessageShareData', 'updateTimelineShareData', 'onMenuShareAppMessage', 'onMenuShareTimeline'], configAppMessage)
					.then((res) => {})
					.catch((res) => {
						if (res.is_ready) {
							res.wx.updateAppMessageShareData(configAppMessage);
							res.wx.updateTimelineShareData(configAppMessage);
							res.wx.onMenuShareAppMessage(configAppMessage);
							res.wx.onMenuShareTimeline(configAppMessage);
						}
					});
			}
		},
		// #endif
		shareH5() {
			// this.giftModalShow = false;
			this.H5ShareBox = true;
		},
		getOrderLottery(status) {
			this.orderLottery = status;
			this.lotteryLoading = true;
		},
		openTap() {
			this.$set(this, 'couponsHidden', !this.couponsHidden);
		},
		onLoadFun: function () {
			this.getOrderPayInfo();
		},
		/**
		 *
		 * 支付完成查询支付状态
		 *
		 */
		getOrderPayInfo: function () {
			let that = this;
			uni.showLoading({
				title: that.$t(`正在加载中`)
			});
			getOrderDetail(that.orderId)
				.then((res) => {
					uni.hideLoading();
					that.$set(that, 'order_pay_info', res.data);
					uni.setNavigationBarTitle({
						title: res.data.paid ? that.$t(`支付成功`) : that.$t(`未支付`)
					});
					this.loading = true;
					if (res.data.paid && res.data.is_gift) {
						this.storeInfo = res.data.cartInfo[0].productInfo;
						this.is_gift = res.data.is_gift;
					}
					this.giftData = {
						image: res.data.cartInfo[0].productInfo.image,
						title: res.data.cartInfo[0].productInfo.store_name,
						message: res.data.gift_mark,
						id: res.data.id,
						avatar: res.data.avatar,
						nickname: res.data.nickname,
						code: res.data.gift_code
					};
					// #ifdef H5
					if (this.is_gift) this.setOpenShare();
					// #endif
					this.getOrderCoupon();
				})
				.catch((err) => {
					this.loading = true;
					uni.hideLoading();
				});
		},
		getOrderCoupon() {
			let that = this;
			orderCoupon(that.orderId).then((res) => {
				that.couponList = res.data;
			});
		},
		/**
		 * 去首页关闭当前所有页面
		 */
		goIndex: function (e) {
			uni.switchTab({
				url: '/pages/index/index'
			});
		},
		// 去参团页面；
		goPink: function (id) {
			uni.navigateTo({
				url: '/pages/activity/goods_combination_status/index?id=' + id
			});
		},
		/**
		 *
		 * 去订单详情页面
		 */
		goOrderDetails: function (e) {
			let that = this;
			// #ifdef MP
			uni.showLoading({
				title: that.$t(`正在加载中`)
			});
			openOrderSubscribe()
				.then((res) => {
					uni.hideLoading();
					uni.redirectTo({
						url: '/pages/goods/order_details/index?order_id=' + that.orderId
					});
				})
				.catch(() => {
					nui.hideLoading();
				});
			// #endif
			// #ifndef MP
			uni.redirectTo({
				url: '/pages/goods/order_details/index?order_id=' + that.orderId
			});
			// #endif
		}
	}
};
</script>

<style lang="scss">
.canvas {
	width: 750rpx;
	height: 1108rpx;
	z-index: 9999;
	position: absolute;
	bottom: 40000rpx;
	right: 30000rpx;
}
.coupons {
	.title {
		margin: 30rpx 0 25rpx 0;
		font-size: 28rpx;
		color: #333333;
	}

	.list {
		padding: 0 20rpx;

		.item {
			margin-bottom: 20rpx;
			box-shadow: 0px 2px 10px 0px rgba(0, 0, 0, 0.06);

			.price {
				width: 236rpx;
				height: 160rpx;
				font-size: 26rpx;
				color: #fff;
				font-weight: 800;

				text {
					font-size: 54rpx;
				}
			}

			.text {
				width: 385rpx;

				.name {
					font-size: #282828;
					font-size: 30rpx;
				}

				.priceMin {
					font-size: 24rpx;
					color: #999;
					margin-top: 10rpx;
				}

				.time {
					font-size: 24rpx;
					color: #999;
					margin-top: 15rpx;
				}
			}
		}

		.open {
			font-size: 24rpx;
			color: #999;
			margin-top: 30rpx;

			.iconfont {
				font-size: 25rpx;
				margin: 5rpx 0 0 10rpx;
			}
		}
	}
}

.payment-status {
	background-color: #fff;
	margin: 195rpx 30rpx 0 30rpx;
	border-radius: 10rpx;
	padding: 1rpx 0 28rpx 0;
}

.payment-status .icons {
	font-size: 70rpx;
	width: 140rpx;
	height: 140rpx;
	border-radius: 50%;
	color: #fff;
	text-align: center;
	line-height: 140rpx;
	text-shadow: 0px 4px 0px rgba(255, 255, 255, 0.5);
	border: 6rpx solid #f5f5f5;
	margin: -76rpx auto 0 auto;
	background-color: #999;
}

.payment-status .icons.icon-iconfontguanbi {
	text-shadow: 0px 4px 0px #6c6d6d;
}

.payment-status .iconfont.fail {
	text-shadow: 0px 4px 0px #7a7a7a;
}

.payment-status .status {
	font-size: 32rpx;
	font-weight: bold;
	text-align: center;
	margin: 25rpx 0 37rpx 0;
}

.payment-status .wrapper {
	border: 1rpx solid #eee;
	margin: 0 30rpx 47rpx 30rpx;
	padding: 35rpx 0;
	border-left: 0;
	border-right: 0;
}

.payment-status .wrapper .item {
	font-size: 28rpx;
	color: #282828;
}

.payment-status .wrapper .item ~ .item {
	margin-top: 20rpx;
}

.payment-status .wrapper .item .itemCom {
	color: #666;
}

.payment-status .returnBnt {
	width: 630rpx;
	height: 86rpx;
	border-radius: 50rpx;
	color: #fff;
	font-size: 30rpx;
	text-align: center;
	line-height: 86rpx;
	margin: 0 auto 20rpx auto;
}
.share-box {
	z-index: 1000;
	position: fixed;
	left: 0;
	top: 0;
	width: 100%;
	height: 100%;

	image {
		width: 100%;
		height: 100%;
	}
}
</style>