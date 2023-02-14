<template>
	<view class="play-song">
		<view class="sing-top">
			<view class="sing-top-name">
				<block v-for="(singer,index) in ar" :key="index">
					<text style="margin-right: 10rpx;" @click="goToSinger(singer.id)">{{singer.name}}</text>
				</block>
			</view>
			<view class="img-box" :class="[isPlay ? '' : 'stoped']">
				<view class="circle">
					<image class="img" :src="musicSrc" mode=""></image>
				</view>

			</view>
		</view>
		<view class="lyric-box">
			<view class="ullrc" :style="{marginTop:mtop+'rpx'}">
				<view class="ric" v-for="(item,index) in lrcList" :key="index" :class="[index==songindex?'cur':'']"
					v-if="lrcList.length>0">
					{{item.words}}
				</view>
				<view class="ric" style="margin-top: 60rpx;">暂无歌词</view>
			</view>
		</view>
		<view class="bottom-control">
			<view class="btn-box">
				<view class="flex-item">
					<text class="iconfont">&#xe615;</text>
				</view>
				<view class="flex-item iconfont">&#xe628;</view>
				<view class="flex-item iconfont">&#xe60e;</view>
			</view>
			<view class="slider-bar">
				<view class="time start">{{currentTime}}</view>
				<slider class="line" :value="currentVal" :min="0" :max="silderVal" @changing="sliderChanging"
					@change="sliderChange" block-size="8" backgroundColor="rgba(255,255,255,.5)"
					activeColor="rgba(255,255,255,.5)" />
				<view class="time end">{{songTime}}</view>
			</view>
			<view class="btn-groups flex-box">
				<view class="flex-item" @click="setPlayModel">
					<view v-if="playModel==0" class="iconfont">&#xe66c;</view>
					<view v-if="playModel==1" class="iconfont">&#xe66b;</view>
					<view v-if="playModel==2" class="iconfont">&#xe66d;</view>
				</view>
				<view class="flex-item" @click="playPrevSong">
					<view class="iconfont">&#xe78a;</view>
				</view>
				<view class="play-btn flex-item" @click="play">
					<view v-if="!isPlay" class="iconfont">&#xe638;</view>
					<view v-if="isPlay" class="iconfont">&#xe76a;</view>
				</view>
				<view class="flex-item" @click="playNextSong">
					<view class="iconfont">&#xe7a5;</view>
				</view>
				<view class="flex-item iconfont" @click="isShowList=true" id="list">&#xe634;</view>
			</view>
		</view>
		<view class="poplist-box" v-if="isShowList">
			<view class="poplist-title">
				<text>当前播放({{songList.length}})首</text>
				<image src="../../static/icons/close-icon.png" mode="" style="width: 20rpx;height: 20rpx;"
					@click="isShowList=false"></image>
			</view>
			<view class="poplist-list">
				<block v-for="(item,index) in songList" :key="item.id">
					<view class="remen-item">
						<image :src="item.al.picUrl" mode=""
							style="width: 80rpx;height: 80rpx;position: absolute;left: 0;border-radius: 10rpx;"
							class="avatar"></image>
						<view class="song-intro" style="margin-left: 100rpx;width: 60%;" @click="playThisSong(item)">
							<view class="song-name" :style="{'color':item.id==songid?'red':''}">
								{{item.name}}
							</view>
							<view class="song-desc" style="">
								<view class="" v-for="(sing,index) in item.ar" :key="index">
									<text :style="{'color':item.id==songid?'red':''}">{{sing.name}}</text>-
								</view>
								<text :style="{'color':item.id==songid?'red':''}">-</text>
								<text :style="{'color':item.id==songid?'red':''}">{{item.al.name}}</text>
							</view>

						</view>

					</view>


				</block>
			</view>
		</view>
	</view>
</template>

<script>
	import {
		mapState,
		mapMutations
	} from 'vuex';
	const innerAudioContext = uni.createInnerAudioContext();
	export default {
		data() {
			return {

				lytop: '222334',
				lycur: '44',
				lybot: '55',
				playModel: 1,
				songId: 0,
				songInfo: [],

				songUrl: '',
				songLrc: '',
				lrcList: [],
				songTime: '',
				silderVal: 0,
				currentVal: 0,
				currentTime: '0:00',
				mtop: 0,
				songindex: 0,
				time: 0,
				songList: [],
				singIndex: 0,
				isShowList: false,
				ar: []
			}
		},
		onLoad(option) {


			this.songId = option.id;
			this.time = option.time
			this.$store.state.songid = this.songId
			this.songList = uni.getStorageSync('songList');
			this.getNextSong()
			this.initMusic();
		},
		computed: {
			...mapState(['songid', 'isPlay', 'musicSrc'])
		},
		methods: {
			sliderChanging(e) {
				innerAudioContext.currentTime = e.detail.value;
				this.$store.state.isPlay = true;
				//console.log(e.detail)
			},
			sliderChange(e) {
				innerAudioContext.currentTime = e.detail.value;
				this.$store.state.isPlay = true;
			},
			//初始化音乐信息
			async initMusic() {
				//console.log(this.$store.state.songList)
				const res = await this.$myRequest({
					url: `/song/detail?ids=${this.songId}`
				});
				const songUrl = await this.$myRequest({
					url: `/song/url?id=${this.songId}`
				});
				// Promise.all([])

				this.songInfo = res.songs;
				// console.log(res)
				this.ar = this.songInfo[0].ar
				this.$store.state.musicSrc = this.songInfo[0].al.picUrl;

				uni.setNavigationBarTitle({
					title: this.songInfo[0].name
				});

				this.songUrl = songUrl.data[0].url;
				innerAudioContext.autoplay = true;
				innerAudioContext.src = this.songUrl;
				this.$store.state.musicUrl = this.songUrl
				if (parseFloat(this.time) > 0) {
					innerAudioContext.currentTime = this.time
				}
				innerAudioContext.onCanplay(() => {
					this.silderVal = innerAudioContext.duration
					this.songTime = '0' + parseInt(innerAudioContext.duration / 60) + ':' + (parseInt(
						innerAudioContext.duration %
						60) >= 10 ? parseInt(innerAudioContext.duration % 60) : '0' + parseInt(
						innerAudioContext.duration % 60));
						this.initSongLrc(this.songId);

				});
				innerAudioContext.onTimeUpdate(() => {
					
					if(!this.isPlay){
						this.$store.state.isPlay = true;
					}

					this.currentVal = innerAudioContext.currentTime;
					this.$store.state.musicCurrentTime = innerAudioContext.currentTime;

					this.currentTime = '0' + parseInt(this.$store.state.musicCurrentTime / 60) + ':' + (
						parseInt(this.$store.state.musicCurrentTime %
							60) >= 10 ? parseInt(this.$store.state.musicCurrentTime % 60) : '0' + parseInt(
							this.$store.state.musicCurrentTime %
							60));
					this.setPosition(this.$store.state.musicCurrentTime);

				});
				innerAudioContext.onEnded(() => {
					this.initNextSong();
					// console.log(songUrl)
				})


			},

			play() {
				console.log(innerAudioContext)
				if (this.$store.state.isPlay) {
					innerAudioContext.pause()
					//console.log('pause')
					this.$store.state.isPlay = false;
					this.$store.state.isPlay = !this.$store.state.isPlay;

				} else {
					//console.log('play')
					innerAudioContext.play()
				}

				this.$store.state.isPlay = !this.$store.state.isPlay
			},
			async initSongLrc(id) {
				const songLrc = await this.$myRequest({
					url: `/lyric?id=${id}`
				})
				//console.log(songLrc)
				if (songLrc.lrc) {
					this.songLrc = songLrc.lrc.lyric;
					this.getLrcArray(this.songLrc)

					this.lrcList = this.getLrcArray(this.songLrc)
				}




			},
			getLrcArray(lrc) {
				//1. ["[00:01.06]难念的经", "[00:03.95]演唱：周华健", ....]
				const parts = lrc.split('\n');
				//2. [{time:1.06, words:"难念的经"}, {time:3.95, words:"演唱：周华健"}]
				for (let i = 0; i < parts.length; i++) {
					parts[i] = this.getLrcObj(parts[i]);
				}
				return parts;
				// "[00:01.06]难念的经"  --->  {time:1.06, words:"难念的经"}

			},
			getLrcObj(content) {
				const twoParts = content.split(']');
				//取出时间
				let time = twoParts[0].substr(1);
				const timeParts = time.split(":");
				const minute = +timeParts[0];
				const seconds = +timeParts[1];
				time = minute * 60 + seconds;
				//取出文字
				const words = twoParts[1];
				return {
					time: time,
					words: words
				}
			},
			getLrcIndex(time) {
				for (let i = 0; i < this.lrcList.length; i++) {
					if (time < this.lrcList[i].time) {
						return i - 1;
					}
				}
			},
			setPosition(time) {

				let i = this.getLrcIndex(time);
				this.songindex = i
				//	console.log(i)
				if (i === -1) {
					return; //最开始的情况，没有命中任何一句歌词，不管它
				}
				let top = i * 46 + 46 / 2 - 180 / 2;

				if (top < 0) {
					top = 0;
				}

				//console.log(this.songindex )
				//console.log(top)
				this.mtop = -top;
			},
			//获取下一首歌曲
			getNextSong() {
				//console.log(this.songList)
				this.singIndex = this.songList.findIndex((item) => {
					return item.id == this.songId
				});
				this.$store.state.currentIndex = this.singIndex

			},
			//播放下一首歌曲
			async playNextSong() {

				if (this.singIndex == this.songList.length - 1) {
					this.singIndex = 0;

					await this.initSong(this.singIndex)

				} else {
				this.$store.state.isPlay = true;
					this.initNextSong();
				}


			},
			//播放上一首歌曲
			playPrevSong() {
				if (this.singIndex == 0) {
					uni.showToast({
						title: '已经是第一首'
					});
					return;
				};
				this.$store.state.isPlay = true;
				this.initPrevSong();
			},
			async initNextSong() {
				this.singIndex += 1;
				this.$store.state.currentIndex = this.singIndex
				this.initSong(this.singIndex)
			},
			async initPrevSong() {

				this.singIndex -= 1;
				this.$store.state.currentIndex = this.singIndex
				this.initSong(this.singIndex)
			},
			async initSongPlay(songId) {
				uni.showLoading({
					title: '加载中'
				});
				const songUrlRes = await this.$myRequest({
					url: `/song/url?id=${songId}`
				});
				const songDetailRes = await this.$myRequest({
					url: `/song/detail?ids=${songId}`
				});
				Promise.all([songUrlRes, songDetailRes]).then(promise => {
					uni.hideLoading()
					console.log(promise)
					const songUrl = promise[0]
					const songDetail = promise[1]
					uni.setNavigationBarTitle({
						title: songDetail.songs[0].name
					});
					this.ar = songDetailRes.songs[0].ar
					this.$store.state.musicSrc = songDetailRes.songs[0].al.picUrl;
					this.$store.state.songid = songId
					innerAudioContext.src = songUrl.data[0].url;
					this.initSongLrc(songId);
				})
			},
			async initSong(index) {
				const nextSongId = this.songList[index].id;
				this.initSongPlay(nextSongId)
			},
			//点击列表播放歌曲
			async playThisSong(option) {
				this.initSongPlay(option.id)
				this.isShowList = false;
			},
			//去歌手信息详情
			goToSinger(id) {
				uni.navigateTo({
					url: '../../components/singerInfo/singerInfo?id=' + id
				})
			}
		}
	}
</script>

<style lang="scss">
	@import "../../common/player.scss";
</style>
