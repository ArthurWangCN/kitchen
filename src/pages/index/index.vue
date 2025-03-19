<template>
  <view class="container">
    <!-- 门店信息 -->
    <view class="store-info">
      <view class="store-header">
        <view class="store-header-left">
          <image class="store-avatar" src="/static/avatar.jpg" mode="aspectFill" />

          <view class="store-meta">
            <text class="store-name">车车小厨房</text>
            <text class="store-time">营业时间：00:00-24:00</text>
            <text class="store-delivery">外卖配送时间：约30分钟</text>
          </view>
        </view>
        <text class="store-status">营业中</text>
      </view>
    </view>

    <!-- 左侧分类列表 -->
    <view class="main-content">
      <scroll-view class="category-list" scroll-y>
        <view v-for="(category, index) in categories" :key="category.id"
          :class="['category-item', { active: currentCategory === category.id }]"
          @tap="selectCategory(category.id, index)">
          {{ category.name }}
        </view>
      </scroll-view>

      <!-- 右侧商品列表 -->
      <scroll-view class="goods-list" scroll-y scroll-with-animation :scroll-top="scrollTop" @scroll="onGoodsScroll">
        <view v-for="category in categories" :key="category.id" :id="'category-' + category.id" class="goods-category">
          <view class="category-title">{{ category.name }}</view>
          <view class="goods-items">
            <view v-for="goods in category.goods" :key="goods.id" class="goods-item">
              <image class="goods-image" :src="goods.image" mode="aspectFill" />
              <view class="goods-info">
                <text class="goods-name">{{ goods.name }}</text>
                <text class="goods-desc">{{ goods.desc }}</text>
                <view class="goods-bottom">
                  <text class="goods-price">¥{{ goods.price }}</text>
                  <view class="goods-action">
                    <view v-if="cartItems.get(goods.id)" class="remove-btn" @tap="removeFromCart(goods.id)">-</view>
                    <text v-if="cartItems.get(goods.id)" class="goods-quantity">{{ cartItems.get(goods.id).quantity
                      }}</text>
                    <view class="add-btn" @tap="addToCart(goods)">+</view>
                  </view>
                </view>
              </view>
            </view>
          </view>
        </view>
      </scroll-view>
    </view>

    <!-- 购物车底栏 -->
    <view v-if="cartItems.size > 0" class="cart-bar" @tap="showCartPopup = true">
      <view class="cart-info">
        <text class="cart-total">合计：¥{{ cartTotal }}</text>
      </view>
      <view class="checkout-btn">去结算</view>
    </view>

    <!-- 购物车弹窗 -->
    <view v-if="showCartPopup" class="cart-popup-mask" @tap="showCartPopup = false">
      <view class="cart-popup" @tap.stop>
        <view class="cart-popup-header">
          <text class="cart-popup-title">购物车</text>
          <text class="cart-popup-close" @tap="showCartPopup = false">×</text>
        </view>
        <scroll-view class="cart-popup-content" scroll-y>
          <view v-for="item in Array.from(cartItems.values())" :key="item.id" class="cart-item">
            <image class="cart-item-image" :src="item.image" mode="aspectFill" />
            <view class="cart-item-info">
              <text class="cart-item-name">{{ item.name }}</text>
              <view class="cart-item-bottom">
                <text class="cart-item-price">¥{{ item.price }}</text>
                <view class="cart-item-action">
                  <view class="remove-btn" @tap="removeFromCart(item.id)">-</view>
                  <text class="goods-quantity">{{ item.quantity }}</text>
                  <view class="add-btn" @tap="addToCart(item)">+</view>
                </view>
              </view>
            </view>
          </view>
        </scroll-view>
        <view class="cart-popup-footer">
          <view class="cart-popup-total">
            <text>合计：</text>
            <text class="cart-popup-price">¥{{ cartTotal }}</text>
          </view>
          <view class="cart-popup-checkout">去结算</view>
        </view>
      </view>
    </view>
  </view>
</template>

<script setup lang="ts">
import { ref, reactive, computed, onMounted } from 'vue'

// 在组件挂载后获取分类位置信息
onMounted(() => {
  setTimeout(() => {
    getCategoryTops()
  }, 100)
})

// 购物车数据
const cartItems = reactive(new Map())
const cartTotal = computed(() => {
  let total = 0
  cartItems.forEach((item) => {
    total += item.price * item.quantity
  })
  return total.toFixed(2)
})

// 商品分类数据
const categories = reactive([
  {
    id: 1,
    name: '热销套餐',
    goods: [
      { id: 1, name: '香辣鸡腿堡套餐', price: 25.8, image: '/static/drink.jpg', desc: '香辣鸡腿堡+薯条+可乐' },
      { id: 2, name: '双层牛肉堡套餐', price: 28.8, image: '/static/drink.jpg', desc: '双层牛肉堡+薯条+可乐' },
      { id: 9, name: '鸡翅套餐', price: 22.8, image: '/static/drink.jpg' },
      { id: 10, name: '巨无霸套餐', price: 32.8, image: '/static/drink.jpg' },
      { id: 11, name: '麦辣鸡腿套餐', price: 26.8, image: '/static/drink.jpg' },
      { id: 12, name: '鱼排套餐', price: 27.8, image: '/static/drink.jpg' }
    ]
  },
  {
    id: 2,
    name: '主食',
    goods: [
      { id: 3, name: '香辣鸡腿堡', price: 15.8, image: '/static/drink.jpg' },
      { id: 4, name: '双层牛肉堡', price: 18.8, image: '/static/drink.jpg' },
      { id: 13, name: '培根芝士堡', price: 17.8, image: '/static/drink.jpg' },
      { id: 14, name: '鳕鱼堡', price: 16.8, image: '/static/drink.jpg' },
      { id: 15, name: '麦香鸡', price: 12.8, image: '/static/drink.jpg' },
      { id: 16, name: '巨无霸', price: 21.8, image: '/static/drink.jpg' }
    ]
  },
  {
    id: 3,
    name: '小吃',
    goods: [
      { id: 5, name: '薯条', price: 8.8, image: '/static/drink.jpg' },
      { id: 6, name: '鸡块', price: 12.8, image: '/static/drink.jpg' },
      { id: 17, name: '洋葱圈', price: 8.8, image: '/static/drink.jpg' },
      { id: 18, name: '玉米杯', price: 7.8, image: '/static/drink.jpg' },
      { id: 19, name: '麦乐鸡', price: 11.8, image: '/static/drink.jpg' },
      { id: 20, name: '甜筒', price: 5.8, image: '/static/drink.jpg' }
    ]
  },
  {
    id: 4,
    name: '饮料',
    goods: [
      { id: 7, name: '可乐', price: 5.8, image: '/static/drink.jpg' },
      { id: 8, name: '雪碧', price: 5.8, image: '/static/drink.jpg' },
      { id: 21, name: '橙汁', price: 6.8, image: '/static/drink.jpg' },
      { id: 22, name: '奶茶', price: 8.8, image: '/static/drink.jpg' },
      { id: 23, name: '咖啡', price: 9.8, image: '/static/drink.jpg' },
      { id: 24, name: '柠檬水', price: 6.8, image: '/static/drink.jpg' }
    ]
  },
])

// 当前选中的分类索引
const currentCategory = ref(1)

// 购物车弹窗显示状态
const showCartPopup = ref(false)

// 分类位置信息
const categoryTops = ref<number[]>([])

// 获取分类位置信息
const getCategoryTops = () => {
  const query = uni.createSelectorQuery()
  categories.forEach((_, index) => {
    query.select(`#category-${index + 1}`).boundingClientRect()
  })
  query.select('.goods-list').boundingClientRect()
  query.exec((res) => {
    const listTop = res[res.length - 1].top
    categoryTops.value = res.slice(0, -1).map((item: any) => item.top - listTop)
  })
}

// 监听滚动事件
const onGoodsScroll = (e: any) => {
  const scrollTop = e.detail.scrollTop
  const index = categoryTops.value.findIndex((top, i) => {
    const nextTop = categoryTops.value[i + 1]
    return scrollTop >= top && (!nextTop || scrollTop < nextTop)
  })
  if (index !== -1) {
    currentCategory.value = index + 1
  }
}

const scrollTop = ref(0)

// 选择分类
const selectCategory = (id: number, index: number) => {
  currentCategory.value = id
  scrollTop.value = categoryTops.value?.[index]
}

// 添加到购物车
const addToCart = (goods: any) => {
  const existingItem = cartItems.get(goods.id)
  if (existingItem) {
    existingItem.quantity++
  } else {
    cartItems.set(goods.id, { ...goods, quantity: 1 })
  }
}

// 从购物车移除
const removeFromCart = (goodsId: number) => {
  const item = cartItems.get(goodsId)
  if (item && item.quantity > 1) {
    item.quantity--
  } else {
    cartItems.delete(goodsId)
  }
}
</script>

<style>
.container {
  display: flex;
  flex-direction: column;
  height: 100vh;
  background-color: #f8f8f8;
  overflow: hidden;
}

.store-info {
  background-color: #fff;
  padding: 20rpx 30rpx;
  margin-bottom: 2rpx;
  box-shadow: 0 2rpx 6rpx rgba(0, 0, 0, 0.05);
}

.store-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 10rpx;
}

.store-header-left {
  display: flex;
  /* flex-direction: column; */
  /* align-items: center; */
  gap: 12rpx;
}

.store-avatar {
  width: 120rpx;
  height: 120rpx;
  border-radius: 16rpx;
  margin-right: 16rpx;
  border: 2rpx solid #eee;
}

.store-name {
  font-size: 34rpx;
  font-weight: bold;
  color: #333;
}

.store-status {
  font-size: 24rpx;
  color: #07c160;
  background: rgba(7, 193, 96, 0.1);
  padding: 4rpx 12rpx;
  border-radius: 6rpx;
}

.store-meta {
  display: flex;
  flex-direction: column;
  gap: 6rpx;
}

.store-time,
.store-delivery {
  font-size: 24rpx;
  color: #666;
}

.main-content {
  display: flex;
  flex: 1;
  overflow: hidden;
  height: calc(100vh - 100rpx - 142rpx);
}

.category-list {
  width: 180rpx;
  height: 100%;
  background-color: #f7f7f7;
  border-right: 2rpx solid #eee;
}

.category-item {
  height: 100rpx;
  line-height: 100rpx;
  text-align: center;
  font-size: 28rpx;
  color: #333;
  border-bottom: 1rpx solid #eee;
}

.category-item.active {
  background-color: #fff;
  color: #FEB2AF;
  font-weight: bold;
}

.goods-list {
  flex: 1;
  height: 100%;
  background-color: #fff;
  /* padding: 0 20rpx; */
}

.goods-category {
  /* padding: 20rpx; */
}

.category-title {
  font-size: 32rpx;
  font-weight: bold;
  padding: 20rpx;
}

.goods-items {
  display: flex;
  flex-direction: column;
}

.goods-item {
  display: flex;
  padding: 24rpx;
  background: #fff;
  border-radius: 16rpx;
  margin-bottom: 20rpx;
}
.goods-item:last-child {
  margin-bottom: 100rpx;
}

.goods-image {
  width: 180rpx;
  height: 180rpx;
  margin-right: 24rpx;
  border-radius: 8rpx;
}

.goods-info {
  flex: 1;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}

.goods-name {
  font-size: 30rpx;
  color: #333;
  font-weight: bold;
  margin-bottom: 8rpx;
}

.goods-desc {
  font-size: 24rpx;
  color: #999;
  margin-bottom: 16rpx;
}

.goods-bottom {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.goods-price {
  font-size: 32rpx;
  color: #FEB2AF;
  font-weight: bold;
}

.goods-action {
  display: flex;
  align-items: center;
}

.add-btn,
.remove-btn {
  width: 50rpx;
  height: 50rpx;
  background-color: #FEB2AF;
  color: #fff;
  border-radius: 25rpx;
  font-size: 38rpx;
  padding: 0;
  display: flex;
  align-items: center;
  justify-content: center;
}

.remove-btn {
  background-color: #999;
}

.goods-quantity {
  font-size: 28rpx;
  margin: 0 20rpx;
}

.cart-bar {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  height: 120rpx;
  background: #fff;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 40rpx;
  box-shadow: 0 -2rpx 10rpx rgba(0, 0, 0, 0.1);
}

.cart-total {
  font-size: 32rpx;
  color: #FEB2AF;
  font-weight: bold;
}

.checkout-btn {
  background: #FEB2AF;
  color: #fff;
  padding: 0 50rpx;
  height: 80rpx;
  line-height: 80rpx;
  border-radius: 40rpx;
  font-size: 28rpx;
  margin: 0;
}

.cart-popup-mask {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  z-index: 999;
}

.cart-popup {
  position: fixed;
  left: 0;
  right: 0;
  bottom: 0;
  background: #fff;
  border-radius: 32rpx 32rpx 0 0;
  z-index: 1000;
  max-height: 80vh;
  display: flex;
  flex-direction: column;
}

.cart-popup-header {
  padding: 40rpx;
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-bottom: 1rpx solid #eee;
}

.cart-popup-title {
  font-size: 32rpx;
  font-weight: bold;
}

.cart-popup-close {
  font-size: 40rpx;
  color: #999;
  padding: 0 20rpx;
}

.cart-popup-content {
  flex: 1;
  max-height: calc(80vh - 200rpx);
}

.cart-item {
  display: flex;
  padding: 30rpx 40rpx;
  border-bottom: 1rpx solid #f5f5f5;
}

.cart-item-image {
  width: 120rpx;
  height: 120rpx;
  border-radius: 8rpx;
  margin-right: 20rpx;
}

.cart-item-info {
  flex: 1;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}

.cart-item-name {
  font-size: 28rpx;
  color: #333;
}

.cart-item-bottom {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.cart-item-price {
  font-size: 30rpx;
  color: #FEB2AF;
  font-weight: bold;
}

.cart-item-action {
  display: flex;
  align-items: center;
}

.cart-popup-footer {
  padding: 20rpx 30rpx;
  border-top: 1rpx solid #eee;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.cart-popup-total {
  font-size: 32rpx;
  font-weight: bold;
  color: #FEB2AF;
}

.cart-popup-price {
  font-size: 32rpx;
  font-weight: bold;
}

.cart-popup-checkout {
  background: #FEB2AF;
  color: #fff;
  padding: 0 50rpx;
  height: 80rpx;
  line-height: 80rpx;
  border-radius: 40rpx;
  font-size: 28rpx;
  margin: 0;
}
</style>
