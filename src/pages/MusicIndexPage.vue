<template>
	<view class="page-wrapper">
		<view class="page-container">
			<MusicHomeComponent v-show="activeIndex===0" />
			<MusicRecommentComponent v-if="isInitComponent[1]" v-show="activeIndex===1"/>
			<MusicCircleComponent v-if="isInitComponent[2]" v-show="activeIndex===2" />
			<MusicMyComponent v-if="isInitComponent[3]" v-show="activeIndex===3"/>
		</view>
		<view class="tab-bar">
			<view class="tab-item" @click="useTab(0)">
				<image class="tab-icon" v-if="activeIndex === 0" src="../../static/icon_home_active.png" />
				<image class="tab-icon" v-else src="../../static/icon_home.png" />
				<text class="tab-text" :class='{"tab-text-active" : activeIndex === 0}'>首页</text>
			</view>
			<view class="tab-item" @click="useTab(1)">
				<image class="tab-icon" v-if="activeIndex === 1" src="../../static/icon_recomment_active.png" />
				<image class="tab-icon" v-else src="../../static/icon_recomment.png" />
				<text class="tab-text" :class='{"tab-text-active" : activeIndex === 1}'>推荐</text>
			</view>
			<view class="mini-player-empty"></view>
			<view class="mini-player-wrapper" @click="useNavigateTo"
				:style="{transform:'translateX(-50%) rotate(' + angle + 'deg)'}">
				<image class="music-img-cover" v-if="store.musicItem?.id" :src="getMusicCover(store.musicItem?.cover)" />
				<image v-else class="music-img-default" src="../../static/icon_music.png" alt="" />
			</view>
			<view class="tab-item" @click="useTab(2)">
				<image class="tab-icon" v-if="activeIndex === 2" src="../../static/icon_music_circle_active.png" />
				<image class="tab-icon" v-else src="../../static/icon_music_circle.png" />
				<text class="tab-text" :class='{"tab-text-active" : activeIndex === 2}'>音乐圈</text>
			</view>
			<view class="tab-item" @click="useTab(3)" data-index="3">
				<image class="tab-icon" v-if="activeIndex === 3" src="../../static/icon_user_active.png" />
				<image class="tab-icon" v-else src="../../static/icon_user.png" />
				<text class="tab-text" :class='{"tab-text-active" : activeIndex === 3}'>我的</text>
			</view>
		</view>
	</view>
</template>

<script setup lang="ts">
	import MusicHomeComponent from '../components/MusicHomeComponent.vue';
	import MusicRecommentComponent from '../components/MusicRecommentComponent.vue';
	import MusicCircleComponent from '../components/MusicCircleComponent.vue';
	import MusicMyComponent from '../components/MusicMyComponent.vue';
	import { useStore } from "../stores/useStore";
	import { ref, onMounted, reactive, onActivated, onDeactivated, onUnmounted } from 'vue';
	import { MUSIC_STORAGE_KEY, MUSIC_LIST_STORAGE_KEY, LOOP_STORAGE_KEY} from '../common/constant';
	import { LoopModeEnum } from '../common/enum';
	import type { MusicType } from '../types';
	import {getMusicCover} from '../utils/util'

	const angle = ref<number>(0);// 旋转的角度
	const store = useStore();
	const activeIndex = ref<number>(0)
	const isInitComponent = reactive<Array<boolean>>([true, false, false, false])
	const useTab = (index : number) => {
		activeIndex.value = index;
		isInitComponent[index] = true
	}

	/**
	 * @description: 头像旋转
	 * @date: 2024-05-10 22:10
	 * @author wuwenqiang
	 */
	const useRotate = () => {
		angle.value += 10
		angle.value = angle.value === 360 ? 0 : angle.value;
	}

	/**
	 * @description: 跳转播放器页面
	 * @date: 2024-05-10 22:10
	 * @author wuwenqiang
	 */
	const useNavigateTo = () => {
		uni.navigateTo({
			url: `../pages/MusicPlayerPage`
		})
	}

	onMounted(() => {
		store.audio.onTimeUpdate(useRotate);
		uni.getStorage({
			key: MUSIC_STORAGE_KEY,
			success: (res) => {
				if (res.data !== '' && res.data !== null) {
					const musicItem:MusicType = JSON.parse(res.data) as MusicType;
					store.setMusic(musicItem, false);
				}
			},
		});

		uni.getStorage({
			key: MUSIC_LIST_STORAGE_KEY,
			success: (res) => {
				if (res.data !== '' && res.data !== null) {
					const musicList: Array<MusicType> = JSON.parse(res.data) as Array<MusicType>;
					store.setMusicList(musicList);
				}
			},
		});

		uni.getStorage({
			key: LOOP_STORAGE_KEY,
			success: (res) => {
				if (res.data !== '' && res.data !== null) {
					store.setLoop(res.data as LoopModeEnum)
				}
			},
		});
	});

	onActivated(() => store.audio.onTimeUpdate(useRotate));// 从缓存中激活

	/**
	 * @description: 移除监听事件
	 * @date: 2024-05-10 22:10
	 * @author wuwenqiang
	 */
	onDeactivated(() => store.audio.offTimeUpdate(useRotate));// 进入缓存

	onUnmounted(() => store.audio.offTimeUpdate(useRotate));// 销毁
</script>

<style lang="less">
	@import '../theme/color.less';
	@import '../theme/size.less';
	@import '../theme/style.less';

	.page-wrapper {
		position: relative;

		.page-container {
			flex: 1;
			height: 0;
			padding: 0 @page-padding;
			box-sizing: border-box;
			overflow: auto;

			&::-webkit-scrollbar {
				display: none;
			}
		}

		.tab-bar {
			z-index: 1;
			position: relative;
			display: flex;
			background-color: @module-background-color;
			border-top: @tab-border-buttom;
			padding: @middle-margin 0;
			position: relative;

			.mini-player-empty {
				width: @mini-music-player-size;
			}

			.mini-player-wrapper {
				position: absolute;
				left: 50%;
				top: -50%;
				transform: translateX(-50%);
				width: @mini-music-player-size;
				height: @mini-music-player-size;
				border-radius: 50%;
				overflow: hidden;
				border: @mini-player-border-size solid @page-background-color;
				background-color: @white-background-color;
				display: flex;
				justify-content: center;
				align-items: center;

				.music-img-default {
					width: @big-icon-size;
					height: @big-icon-size;
				}

				.music-img-cover {
					width: 100%;
					height: 100%;
				}
			}

			.tab-item {
				flex: 1;
				align-items: center;
				justify-content: center;
				display: flex;
				flex-direction: column;

				.tab-icon {
					width: @middle-icon-size;
					height: @middle-icon-size;
					margin-bottom: @small-margin;
				}

				.tab-text {
					color: #000;
					font-size: @font-size-normal;

					&.tab-text-active {
						color: @tab-color-active;
					}
				}
			}
		}
	}
</style>