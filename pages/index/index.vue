<template>
	<view class="avatar-mask-page">
    <!-- 背景图 -->
    <image class="background" mode="aspectFill" src="/static/images/top-bg.png"></image>
<!--  页面内容  -->
    <view class="header" id="avatar-container" @touchstart="touchStart" @touchend="touchEnd" @touchmove="touchMove">
      <!-- 图片容器 -->
      <view class="content"  @touchstart="touchAvatarBg">
        <!--    默认头像      -->
        <image class="avatar" id="avatar-bg" :src="avatarPath"></image>
        <!-- canvas容器 用于生成图片保存-->
        <canvas class="cans-id-mask" canvas-id="cans-id-mask" style="width: 128px; height: 125px;"/>
        <!--    获取微信头像      -->
		<button id="btn-my-avatar" class="get-avatar" open-type="chooseAvatar" @chooseavatar="getUserInfoCallBack">获取微信头像</button>
      </view>
      <!--    操作按钮    -->
      <view class="operation-container">
        <button class="operation" @click="chooseImage()">选择图片</button>
        <button class="operation" @click="draw()">保存头像</button>
      </view>
      <!-- 贴图 -->
      <image v-if="currentMaskId > -1" class="mask flip-horizontal" :class="{maskWithBorder: showBorder}" id='mask' :src="maskPic"
             :style="{top: maskCenterY-maskSize.h/2-2+'px', left: maskCenterX-maskSize.w/2-2+'px',transform: 'rotate(' +rotate+ 'deg)' + 'scale(' +scale+')' + 'rotateY('+ rotateY +'deg)'}"></image>
      <!-- 缩放、旋转图标 -->
      <text class="cuIcon-full handle circle":class="{hideHandle: !showBorder}" id="handle":style="{top:handleCenterY-10 + 'px', left:handleCenterX-10 +'px'}"></text>
      <!-- 水平翻转图标 -->
      <text class="cuIcon-order cancel circle"v-if="isAndroid":class="{hideHandle: !showBorder}" id="cancel" @click="flipHorizontal":style="{top:cancelCenterY-10 + 'px', left:cancelCenterX-10 +'px', transform: 'rotate(' +90+ 'deg)'}"></text>

    </view>
    <!-- 广告容器 -->
    <!-- <view class="grid justify-around share-wrapper">
      <ad unit-id="adunit-85230d6cd9a1beee"></ad>
    </view> -->
    <!-- 图片列表数据 -->
    <view class="resources">
<!--   图片列表   -->
      <ul class="tabs">
        <li class="tab-item" :class="{active: item.id == selectedTab}"
		v-for="(item,index) in tabList" :key="item.id" @tap="switchTab(item)">{{item.title}}</li>
      </ul>
	<ul class="tab-content">
	  <view class="mask-item" v-for="(item,index) in imgList" :key="index">
		<image :src="'/static/images/'+ selectedTab +'/'+ index +'.png'" :data-mask-id="index" @tap="changeMask"></image>
	  </view>
	</ul>
     
    </view>
	</view>
</template>
<script>
	// 在页面中定义激励视频广告
	let videoAd = null;
	// 在页面中定义插屏广告
	let interstitialAd = null

	const range = (start, end, step) => {
		return Array.from(Array.from(Array(Math.ceil((end - start) / step)).keys()), x => start + x * step);
	}
	const STORAGE_KEY = 'PLUG-ADD-MYAPP-KEY';

	export default {
		components: {
		},
		data() {
			return {
				SHOW_TIP: false,
				duration: 15,
				statusBarHeight: 0,
				windowHeight: 0,
				isAndroid: getApp().globalData.IS_ANDROID,
				modalName: null,
				savedCounts: 0,
				cansWidth: 128, // 宽度 px
				cansHeight: 125, // 高度 px
				avatarPath: '/static/images/default.png',
				
				currentMaskId: -1,
				showBorder: false,
				maskCenterX: wx.getSystemInfoSync().windowWidth / 2,
				maskCenterY: 250,
				cancelCenterX: wx.getSystemInfoSync().windowWidth / 2 - 32.5 - 2,
				cancelCenterY: 180,
				handleCenterX: wx.getSystemInfoSync().windowWidth / 2 + 32.5 - 2,
				handleCenterY: 270,
				maskSize: {
				  w: 65,
				  h: 40
				},
				scale: 1,
				rotate: 0,
				rotateY: 0, // 值180时，则水平翻转
				mask_center_x: wx.getSystemInfoSync().windowWidth / 2,
				mask_center_y: 250,
				cancel_center_x: wx.getSystemInfoSync().windowWidth / 2 - 32.5 - 2,
				cancel_center_y: 180,
				handle_center_x: wx.getSystemInfoSync().windowWidth / 2 + 32.5 - 2,
				handle_center_y: 270,
				scaleCurrent: 1,
				rotateCurrent: 0,
				touch_target: "",
				start_x: 0,
				start_y: 0,
				tabList: [
					{
						id: 'tiger',
						title: '虎虎生威',
						count: 4
					},
					{
						id: 'mask',
						title: '口罩',
						count: 18
					},
					{
						id: 'flag-bg',
						title: '国旗背景',
						count: 0
					},
					{
						id: 'flag-avatar',
						title: '国旗头像',
						count: 0
					}
				],
				selectedTab: 'mask',
				imgList: range(0, 18, 1), // 第二个参数是个数
			}
		},
		computed: {
			// ...mapState({
			// 	userInfo: 'userInfo'
			// }),
			maskPic: function() {
				return `/static/images/${this.selectedTab}/${this.currentMaskId}.png`;
			}
		},
		onLoad(option) {
			this.windowHeight = getApp().globalData.windowHeight;
			if (!!getApp().globalData.userAvatarFilePath) {
				this.avatarPath = getApp().globalData.userAvatarFilePath;
			}

			let _this = this;

			// 在页面onLoad回调事件中创建插屏广告实例
			// if (wx.createInterstitialAd) {
			// 	interstitialAd = wx.createInterstitialAd({
			// 		adUnitId: 'adunit-2bf7cf186785bfda'
			// 	})
			// 	interstitialAd.onLoad(() => {})
			// 	interstitialAd.onError((err) => {
			// 		console.log(err);
			// 	})
			// 	interstitialAd.onClose(() => {})
			// }

			// 在页面onLoad回调事件中创建激励视频广告实例
			// if (wx.createRewardedVideoAd) {
			// 	videoAd = wx.createRewardedVideoAd({
			// 		adUnitId: 'adunit-9a8af70b40e15f29'
			// 	})
			// 	videoAd.onLoad(() => {
			// 		_this.rewardedVideoAdLoaded = true;
			// 	})
			// 	videoAd.onError((err) => {
			// 		// 广告组件出现错误，直接允许用户保存，不做其他复杂处理
			// 		_this.rewardedVideoAdLoaded = false;
			// 	})
			// 	videoAd.onClose((res) => {
			// 		if (res && res.isEnded || res === undefined) {
			// 			// 正常播放结束，下发奖励
			// 			_this.rewardedVideoAdAlreadyShow = true; // 本次使用不再展现激励广告
			// 			_this.saveCans();
			// 		} else {
			// 			// 播放中途退出，进行提示
			// 			_this.rewardedVideoAdAlreadyShow = false;
			// 			uni.showToast({
			// 				title: '请完整观看哦'
			// 			})
			// 		}

			// 	})
			// }
		},
		onReady() {
			// 判断是否已经显示过
			let cache = uni.getStorageSync(STORAGE_KEY);
			console.log('cache', cache);
			if (!cache) {
				this.statusBarHeight = uni.getSystemInfoSync().statusBarHeight;
				console.log('this.statusBarHeight', this.statusBarHeight);

				// 没显示过，则进行展示
				this.SHOW_TIP = true;
				console.log('SHOW_TIP', this.SHOW_TIP);
				// 关闭时间
				let _this = this;
				setTimeout(() => {
					_this.SHOW_TIP = false;
				}, _this.duration * 1000);
			}
		},
		onShow() {
			console.log("onShow");

			// 在适合的场景显示插屏广告
			if (interstitialAd) {
			  interstitialAd.show().catch((err) => {
			    console.error(err)
			  })
			}

			if (getApp().globalData.rapaintAfterCrop) {
				getApp().globalData.rapaintAfterCrop = false;
				this.avatarPath = getApp().globalData.cropImageFilePath;
				this.paint();
			}
		},
		onShareAppMessage() {
			return {
				title: '我换上了口罩头像，防止疫情蔓延，30款口罩、护目镜任你选！',
				desc: '防传染、戴口罩，从我做起！',
				imageUrl: '/static/images/mask/avatar_mask.png',
				path: '/pages/index/index',
				success: function(res) {
					console.log(res);
				}
			}
		},
		methods: {
			// ...mapMutations(["saveLoginUserInfo"]),
			paint() {},
			showTips() {
				console.log('tips');
				this.modalName = 'tips';
			},
			touchAvatarBg() {
				this.showBorder = false;
			},
			touchStart(e) {
				console.log('e.target.id', e.target.id);
				if (e.target.id == "mask") {
					this.touch_target = "mask";
					this.showBorder = true;
				} else if (e.target.id == "handle") {
					this.touch_target = "handle"
				} else {
					this.touch_target = ""
				};

				if (this.touch_target != "") {
					this.start_x = e.touches[0].clientX;
					this.start_y = e.touches[0].clientY;
				}
			},
			touchEnd(e) {
				this.mask_center_x = this.maskCenterX;
				this.mask_center_y = this.maskCenterY;
				this.cancel_center_x = this.cancelCenterX;
				this.cancel_center_y = this.cancelCenterY;
				this.handle_center_x = this.handleCenterX;
				this.handle_center_y = this.handleCenterY;
				this.touch_target = "";
				this.scaleCurrent = this.scale;
				this.rotateCurrent = this.rotate;
			},
			touchMove(e) {
				var current_x = e.touches[0].clientX;
				var current_y = e.touches[0].clientY;
				var moved_x = current_x - this.start_x;
				var moved_y = current_y - this.start_y;
				if (this.touch_target == "mask") {
					this.maskCenterX = this.maskCenterX + moved_x;
					this.maskCenterY = this.maskCenterY + moved_y;
					this.cancelCenterX = this.cancelCenterX + moved_x;
					this.cancelCenterY = this.cancelCenterY + moved_y;
					this.handleCenterX = this.handleCenterX + moved_x;
					this.handleCenterY = this.handleCenterY + moved_y;
				};
				if (this.touch_target == "handle") {
					this.handleCenterX = this.handleCenterX + moved_x;
					this.handleCenterY = this.handleCenterY + moved_y;
					this.cancelCenterX = 2 * this.maskCenterX - this.handleCenterX;
					this.cancelCenterY = 2 * this.maskCenterY - this.handleCenterY;
					let diff_x_before = this.handle_center_x - this.mask_center_x;
					let diff_y_before = this.handle_center_y - this.mask_center_y;
					let diff_x_after = this.handleCenterX - this.mask_center_x;
					let diff_y_after = this.handleCenterY - this.mask_center_y;
					let distance_before = Math.sqrt(diff_x_before * diff_x_before + diff_y_before * diff_y_before);
					let distance_after = Math.sqrt(diff_x_after * diff_x_after + diff_y_after * diff_y_after);
					let angle_before = Math.atan2(diff_y_before, diff_x_before) / Math.PI * 180;
					let angle_after = Math.atan2(diff_y_after, diff_x_after) / Math.PI * 180;

					this.scale = distance_after / distance_before * this.scaleCurrent;
					this.rotate = angle_after - angle_before + this.rotateCurrent;
				}
				this.start_x = current_x;
				this.start_y = current_y;
			},
			/**
			 *  获取用户信息回调方法
			 * @param {Object} result
			 */
			getUserInfoCallBack(result) {
				const { avatarUrl } = result.detail;
				 if (!avatarUrl) {
					uni.showModal({
						title: '获取用户头像失败',
						content: '用户信息仅用于创建新的图片，请放心使用',
						showCancel: false
					});
					return;
				 }
				getApp().globalData.userAvatarUrl = avatarUrl;
				uni.showLoading({
					title: '头像加载中...'
				});
				let self = this;
				uni.downloadFile({
					url: avatarUrl,
					success: function(res) {
						uni.hideLoading();
						self.avatarPath = res.tempFilePath;
						getApp().globalData.userAvatarFilePath = res.tempFilePath;
					},
					fail: function(e) {
						console.log(e);
						uni.hideLoading();
						uni.showModal({
							title: '图片加载超时',
							content: '检查网络，点击确定重新加载',
							success(res) {
								if (res.confirm) {
									self.downloadAvatarAndPaintAll(imageUrl);
								} else if (res.cancel) {
									console.log('用户点击取消');
								}
							}
						})
					}
				})
			},


			/**
			 *  选择图片
			 */
			chooseImage() {
				let self = this;
				uni.chooseImage({
					count: 1, // 默认9
					sizeType: ['compressed'], //可以指定是原图还是压缩图，默认二者都有
					sourceType: ['album', 'camera'],
					success: function(res) {
					  console.log(res)
						let tempImagePath = res.tempFilePaths[0];
						self.imageCheck(tempImagePath, self.loadRecImageOrStartToCrop);
					}
				});
			},
			loadRecImageOrStartToCrop(tempImagePath) {
				this.ownImageUsed = true;
				let self = this;
				uni.getImageInfo({
					src: tempImagePath,
					success: function(image) {
						let width = image.width;
						let height = image.height;
						let delta = (width - height) / width.toFixed(3);
						// console.log('delta', delta);
						// 如果是正方形或者接近正放心则直接加载不进行剪裁
						// if ((-0.02 <= delta && delta <= 0) || (0 < delta && delta <= 0.02)) {
						// 	let tempFilePathCompressed = tempImagePath;
						// 	self.avatarPath = tempImagePath;
						// 	self.paint();
						// } else {
						// 	uni.navigateTo({
						// 		url: '/pages/crop/crop?tempFilePath=' + tempImagePath
						// 	})
						// }
						// let tempFilePathCompressed = tempImagePath;
						self.avatarPath = tempImagePath;
						self.paint();
					}
				});
			},
			changeMask(e) {
				this.currentMaskId = e.target.dataset.maskId;
				this.showBorder = true;
			},
			switchTab(item) {
				this.selectedTab = item.id;
				// 重置图片
				this.imgList = range(0, item.count, 1)
			},
			draw() {
				let scale = this.scale;
				let rotate = this.rotate;
				let mask_center_x = this.mask_center_x;
				let mask_center_y = this.mask_center_y;
				let _this = this;
				// 创建节点选择器
				// 口罩中心位置的计算是从屏幕左上角开始，所以我们需要获取头像图片的位置，来得到口罩相对头像的位置
				var query = wx.createSelectorQuery();
				query.select('#avatar-bg').boundingClientRect()
				query.exec(function(res) {
					//res就是 所有标签为#的元素的信息的数组
					mask_center_x = mask_center_x - res[0].left;
					mask_center_y = mask_center_y - res[0].top;
					const pc = wx.createCanvasContext('cans-id-mask');
					const maskWidth = _this.maskSize.w * scale;
					const maskHeight = _this.maskSize.h * scale

					pc.clearRect(0, 0, _this.cansWidth, _this.cansHeight);
					pc.drawImage(_this.avatarPath, 0, 0, _this.cansWidth, _this.cansHeight);
					pc.translate(mask_center_x, mask_center_y);
					pc.rotate(rotate * Math.PI / 180);
					if (_this.isAndroid) {
						_this.rotateY == 180 && pc.scale(-1, 1);
					}
					pc.drawImage(_this.maskPic, -maskWidth / 2, -maskHeight/2, maskWidth, maskHeight);
					pc.draw();

					// 有成功加载的激励视频，才展现提示框
					// if (!!videoAd && _this.rewardedVideoAdLoaded) {
					// 	uni.showModal({
					// 		title: '获取无限制使用',
					// 		content: '观看完视频可以自动保存哦',
					// 		success: function(res) {
					// 			if (res.confirm) {
					// 				console.log('用户点击确定');
					// 				// 用户触发广告后，显示激励视频广告
					// 				if (videoAd) {
					// 					_this.rewardedVideoAdAlreadyShow = true;
					// 					videoAd.show().catch(() => {
					// 						// 失败重试
					// 						videoAd.load()
					// 							.then(() => {
					// 								videoAd.show();
					// 							})
					// 							.catch(err => {
					// 								console.log(err);
					// 								console.log('激励视频 广告显示失败')
					// 							})
					// 					})
					// 				}
					// 			} else if (res.cancel) {
					// 				console.log('用户点击取消');
					// 				_this.saveCans();
					// 				return;
					// 			}
					// 		}
					// 	});
					// } else {
					// 	_this.saveCans();
					// }
					_this.saveCans();
				})
			},
			flipHorizontal() {
				this.rotateY = this.rotateY == 0 ? 180 : 0;
			},
			/**
			 * 保存
			 */
			saveCans() {
				let _this = this;
				uni.showLoading({
					title: '保存...',
					mask: true
				})
				uni.canvasToTempFilePath({
					x: 0,
					y: 0,
					height: this.cansWidth,
					width: this.cansHeight,
					destWidth: this.cansWidth * 3,
					destHeight: this.cansHeight * 3,
					canvasId: 'cans-id-mask',
					success: function(res) {
						uni.hideLoading();
						getApp().globalData.maskAvatarSavedTempPath = res.tempFilePath;
						uni.saveImageToPhotosAlbum({
							filePath: res.tempFilePath,
							success: function(res) {
								if (_this.savedCounts == 0) {
									_this.modalName = 'saveTip';
								} else {
									uni.showToast({
										title: '请至相册查看'
									})
								}
								_this.savedCounts++;
								uni.vibrateShort({
									success: function() {
										console.log('vibrateShort');
									}
								});
							},
							fail(res) {
								console.log(res)
								if (res.errMsg.indexOf("fail")) {
									uni.showModal({
										title: '您需要授权相册权限',
										success(res) {
											if (res.confirm) {
												uni.openSetting({
													success(res) {
														console.log("相册授权成功");
													},
													fail(res) {
														console.log(res)
													}
												})
											}
										}
									})
								}
							}
						});
					},
					fail(res) {
						uni.hideLoading()
					}
				}, this)
			},
			showModal: function(e) {
				this.modalName = e.currentTarget.dataset.target;
			},
			hideModal: function(e) {
				this.modalName = null;
			},
			imageCheck: function(tempImagePath, callback) {
				if (!getApp().globalData.enableSecurityCheck) {
					callback(tempImagePath);
					return;
				}
				let _this = this;
				uni.compressImage({
					src: tempImagePath,
					quality: 1,
					success: res => {
						let tempFilePathCompressed = res.tempFilePath;
						wx.getFileSystemManager().readFile({
							filePath: tempFilePathCompressed, // 压缩图片，然后安全检测
							success: buffer => {
								uni.showLoading({
									title: '加载中...'
								});
								callback(tempImagePath);
							}
						})

					}
				})
			}
		}
	}
</script>

<style  scoped>
  page{
    width: 100%;
    height: 100%;
    background: rgba(241, 241, 241,  1);
  }
  .avatar-mask-page{
    width: 100%;
    height: 100%;
    position: relative;
  }
  .avatar-mask-page .background{
    width: 100%;
    height: 466px;
    position: absolute;
    z-index: 1;
    left: 0;
    top: 0;
  }
  .avatar-mask-page .header{
    width: 100%;
    position: relative;
    z-index: 10;
    left: 0;
    top: 0;
    padding-top: 124px;
  }
  .header .content {
    width: 220px;
    height: 220px;
    border-radius: 10px;
    background: #FFFFFF;
    border-radius: 12px;
    margin: auto;
    position: relative;
	overflow: hidden;
  }
  .header .content .avatar {
    margin: 0 auto;
    width: 128px;
    height: 125px;
    display: flex;
    position: relative;
	top: 33px;
  }
  .header .content .cans-id-mask {
    width: 100%;
    height: 100%;
    position: absolute;
   top: 2000px;
   left: 1000px;
  }
  
  .get-avatar{
    width: 124px;
    height: 38px;
    font-size: 14px;
    font-weight: 500;
    color: #333333;
    background: #FFFFFF;
    border-radius: 19px;
    border: 1px solid #DEDEDE;
    margin: 50px auto 0 auto;
  }
  .operation-container{
    display: flex;
    margin: 20px auto 0 auto;
    width: 226px;
  }
  .operation-container .operation{
    width: 96px;
    height: 38px;
    background: #FFFFFF;
    border-radius: 19px;
    border: 1px solid #D9D9D9;
    padding: 0;
    font-size: 14px;
    font-weight: 500;
    color: #333333;
  }
  .operation-container .margin{
    margin: 20rpx 0;
  }
  .avatar-mask-page .resources{
    background: #FFFFFF;
    border-radius: 12px 12px 0px 0px;
    margin-top: 58px;
    position: relative;
    z-index: 10;
    padding: 12px;
  }
  .avatar-mask-page .resources .tabs{
    display: flex;
    flex-wrap: wrap;
    justify-content: space-between;
  }
  .avatar-mask-page .resources .tabs .tab-item{
    font-size: 12px;
    font-weight: 400;
    color: #333333;
    line-height: 17px;
  }
  .avatar-mask-page .resources .tabs .tab-item.active{
    font-size: 14px;
    font-weight: 600;
    color: #FF5C6C;
  }
  .avatar-mask-page .resources .tab-content{
    display: flex;
    flex-wrap: wrap;
    justify-content: space-between;
  }
  .avatar-mask-page .resources .tab-content .mask-item{
    display: inline-flex;
    width: 75px;
    height: 75px;
    background: #FFEBEA;
    border-radius: 8px;
    margin-top: 12px;
  }
  .avatar-mask-page .resources .tab-content .mask-item image {
    height: 40px;
    width: 65px;
    margin: auto;
  }
  
  .action-wrapper {
    padding-top: 10rpx;
    padding-left: 100rpx;
    padding-right: 100rpx;
    font-weight: 800;
  }
  
  .share-wrapper {
    padding-top: 10rpx;
    padding-left: 100rpx;
    padding-right: 100rpx;
    font-weight: 800;
  }
  
  .mask {
    height: 40px;
    width: 65px;
    position: absolute;
    top: 100px;
    border: 3px solid rgba(255, 255, 255, 0.0);
    z-index: 1;
  }
  
  .maskWithBorder {
    border: dashed 1px black;
  }
  
  .hideHandle {
    display: none;
  }
  
  .circle {
    border-radius: 50%;
    font-size: 15px;
    color: #000;
    line-height: 25px;
    text-align: center;
    background: #000;
  }
  
  .handle,
  .cancel {
    position: absolute;
    z-index: 1;
    width: 25px;
    height: 25px;
    background-color: black;
    color: black;
  }
  
  .scrollView {
    width: 100%;
    position: absolute;
    bottom: 0px;
    white-space: nowrap;
  }
  
  .infoView {
    width: 95%;
    position: absolute;
    bottom: 85px;
    white-space: nowrap;
    background-color: white;
    margin: 10px;
    padding: 1px 5px;
    border-radius: 5px;
    white-space: pre-wrap;
  }
  
  .flip-horizontal {
    -moz-transform: scaleX(-1);
    -webkit-transform: scaleX(-1);
    -o-transform: scaleX(-1);
    transform: scaleX(-1);
  }

</style>
