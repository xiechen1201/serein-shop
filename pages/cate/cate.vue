<template>
	<my-search></my-search>
	<view class="scroll-view-container" :style="{ height: wh + 'px' }">
		<!-- 左侧的滚动视图区域 -->
		<scroll-view class="left-scroll-view" scroll-y :style="{ height: wh + 'px' }">
			<block v-for="(item, index) in cateList" :key="item.cat_id">
				<view :class="['left-scroll-view-item', index === active ? 'active' : '']" @click="handleClickCateItem(index)">{{ item.cat_name }}</view>
			</block>
		</scroll-view>
		<!-- 右侧的滚动视图区域 -->
		<scroll-view class="right-scroll-view" scroll-y :style="{ height: wh + 'px' }">
			<block v-for="item in cateLevel2" :key="item.cat_id">
				<view class="cate-lv2-title">「{{ item.cat_name }}」</view>
				<view class="cate-lv3-list">
					<view class="cate-lv3-item" v-for="(item3, i3) in item.children" :key="i3" @click="handleClickGoodsList(item3)">
						<!-- src="/static/load_error.png" -->
						<image :src="item3.cat_icon"></image>
						<text>{{ item3.cat_name }}</text>
					</view>
				</view>
			</block>
		</scroll-view>
	</view>
</template>

<script setup>
import { setBadge } from '../../hooks/useCart.js';

setBadge();
</script>

<script>
export default {
	data() {
		return {
			wh: 0,
			active: 0,
			cateList: [],
			cateLevel2: []
		};
	},
	onLoad() {
		// 获取当前系统的信息
		const sysInfo = uni.getSystemInfoSync();
		// 为 wh 窗口可用高度动态赋值
		// 窗口的可用高度 = 屏幕高度 - navigationBar高度 - tabBar 高度
		this.wh = sysInfo.windowHeight - 50;

		this.getCateList();
	},
	methods: {
		async getCateList() {
			const { data: res } = await uni.$http.get('/api/public/v1/categories');

			if (res.meta.status !== 200) return uni.$showMsg();
			this.cateList = res.message;
			this.cateLevel2 = res.message[0].children;
		},
		handleClickCateItem(index) {
			this.active = index;
			this.cateLevel2 = this.cateList[index].children;
		},
		handleClickGoodsList(item) {
			uni.navigateTo({
				url: '/subpkg/goods_list/goods_list?cid=' + item.cat_id
			});
		}
	}
};
</script>

<style lang="scss">
.scroll-view-container {
	display: flex;

	.left-scroll-view {
		width: 120px;

		.left-scroll-view-item {
			line-height: 60px;
			background-color: #f7f7f7;
			text-align: center;
			font-size: 12px;

			// 激活项的样式
			&.active {
				background-color: #ffffff;
				position: relative;

				// 渲染激活项左侧的红色指示边线
				&::before {
					content: ' ';
					display: block;
					width: 3px;
					height: 30px;
					background-color: #c00000;
					position: absolute;
					left: 0;
					top: 50%;
					transform: translateY(-50%);
				}
			}
		}
	}

	.right-scroll-view {
		.cate-lv2-title {
			font-size: 12px;
			font-weight: bold;
			text-align: center;
			padding: 15px 0;
		}

		.cate-lv3-list {
			display: flex;
			flex-wrap: wrap;

			.cate-lv3-item {
				width: 33.33%;
				margin-bottom: 10px;
				display: flex;
				flex-direction: column;
				align-items: center;

				image {
					width: 60px;
					height: 60px;
				}

				text {
					font-size: 12px;
				}
			}
		}
	}
}
</style>
