<template>
	<view class="content">
		<view class="title">上传稍后关闭,源码有效</view>
		<view class="title"><uni-link href="https://545c.com/d/19473836-42522931-e58c84" text="源码下载(密码2233)"></uni-link></view>
		
		<view class="btn-list">
			<button type="primary" @click="upload">上传图片</button>
			<input type="text" id="url" :value="imgUrl"/>

		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				imgUrl: ''
			}
		},
		methods: {
			upload() {
				new Promise((resolve, reject) => {
					uni.chooseImage({
						count: 1,
						success: res => {
							const path = res.tempFilePaths[0]
							let ext
							// #ifdef H5
							ext = res.tempFiles[0].name.split('.').pop()
							const options = {
								filePath: path,
								cloudPath: Date.now() + '.' + ext
							}
							resolve(options)
							// #endif
							// #ifndef H5
							uni.getImageInfo({
								src: path,
								success(info) {
									const options = {
										filePath: path,
										cloudPath: Date.now() + '.' + info.type.toLowerCase()
									}
									resolve(options)
								},
								fail(err) {
									reject(new Error(err.errMsg || '未能获取图片类型'))
								}
							})
							// #endif
						},
						fail: () => {
							reject(new Error('Fail_Cancel'))
						}
					})
				}).then((options) => {
					uni.showLoading({
						title: '文件上传中...'
					})
					return uniCloud.uploadFile({
						...options,
						onUploadProgress(e) {
							console.log(e)
						}
					})
				}).then(res => {
					uni.hideLoading()
					console.log(res);
					this.fileID = res.fileID
					this.imgUrl = this.fileID
					// uni.showModal({
					// 	content: '图片上传成功，fileId为：' + res.fileID,
					// 	showCancel: false
					// })
				}).catch((err) => {
					uni.hideLoading()
					console.log(err);
					if (err.message !== 'Fail_Cancel') {
						uni.showModal({
							content: `图片上传失败，错误信息为：${err.message}`,
							showCancel: false
						})
					}
				})
			},
			showImg() {
				if (!this.fileID) {
					uni.showModal({
						content: '请先上传一张图片',
						showCancel: false
					})
					return
				}
				// 腾讯云需要获取临时链接
				// uniCloud.getTempFileURL({
				// 	fileList: [this.fileID],
				// 	success: (res) => {
				// 		this.imgUrl = res.fileList[0].tempFileURL
				// 		console.log(res);
				// 	},
				// 	fail: () => {
				// 		uni.showModal({
				// 			content: '获取临时链接失败',
				// 			showCancel: false
				// 		})
				// 	}
				// })
				// 阿里云可以直接使用fileID进行展示
				this.imgUrl = this.fileID
			}
		}
	}
</script>

<style>
	.content {
		padding-bottom: 30px;
	}

	.title {
		font-weight: bold;
		text-align: center;
		padding: 20px 0px;
		font-size: 20px;
	}

	.tips {
		color: #999999;
		font-size: 14px;
		padding: 20px 30px;
	}

	.btn-list {
		padding: 0px 30px;
	}

	.btn-list button {
		margin-bottom: 20px;
	}

	.upload-preview {
		width: 100%;
	}
</style>
