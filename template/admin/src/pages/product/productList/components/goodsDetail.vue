<template>
  <div class="goods_detail">
    <div class="goods_detail_wrapper" style="height: 640px">
      <HappyScroll size="5" resize hide-horizontal>
        <div style="width: 375px">
          <div class="title-box">商品详情</div>
          <div class="swiper-box">
            <el-carousel :autoplay="goodsInfo.productInfo.slider_image.length > 1" v-model="value2" loop arrow="never">
              <el-carousel-item v-for="(item, index) in goodsInfo.productInfo.slider_image" :key="index">
                <div class="demo-carousel"><img :src="item" alt="" /></div>
              </el-carousel-item>
            </el-carousel>
          </div>
          <div class="goods_info">
            <div class="number-wrapper">
              <div class="price"><span>¥</span>{{ goodsInfo.productInfo.price }}</div>
              <div class="old-price" v-if="goodsInfo.productInfo.vip_price != '0.00'">
                ¥{{ goodsInfo.productInfo.vip_price }}
              </div>
              <div v-if="goodsInfo.productInfo.vip_price != '0.00'">
                <img src="../../../../assets/images/goods_vip.png" />
              </div>
            </div>
            <div class="name">{{ goodsInfo.productInfo.store_name }}</div>
            <div class="msg">
              <div class="item">原价:￥{{ goodsInfo.productInfo.ot_price }}</div>
              <div class="item">库存:{{ goodsInfo.productInfo.stock }}{{ goodsInfo.productInfo.unit_name }}</div>
              <div class="item">
                销量:{{ goodsInfo.productInfo.sales + goodsInfo.productInfo.ficti
                }}{{ goodsInfo.productInfo.unit_name }}
              </div>
            </div>
          </div>
          <div class="con-box">
            <div class="title-box">商品介绍</div>
            <div class="content" v-html="goodsInfo.productInfo.description"></div>
          </div>
        </div>
      </HappyScroll>
    </div>
  </div>
</template>

<script>
import { HappyScroll } from 'vue-happy-scroll';
import { productInfoApi } from '@/api/product';
export default {
  name: 'goods_detail',
  props: {
    goodsId: {
      type: String | Number,
      default: '',
    },
  },
  components: {
    HappyScroll,
  },
  data() {
    return {
      value2: 0,
      goodsInfo: {},
    };
  },
  mounted() {
    this.getInfo();
  },
  methods: {
    getInfo() {
      productInfoApi(this.goodsId).then((res) => {
        this.goodsInfo = res.data;
      });
    },
  },
};
</script>

<style lang="scss" scoped>
.goods_detail {
  .goods_detail_wrapper {
    background: url('../../../../assets/images/phonebg.png') no-repeat center top !important;
    background-size: 375px 640px !important;
    padding: 50px 20px;
    z-index: 20;
    position: fixed;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
    width: 375px;
    background: #f0f2f5;
  }
  .title-box {
    width: 335px;
    height: 46px;
    line-height: 46px;
    background: #fff;
    text-align: center;
    color: #333;
    font-size: 16px;
  }
  .swiper-box {
    .demo-carousel {
      width: 333px;

      img {
        width: 100%;
        height: 100%;
        display: block;
      }
    }
  }
  .goods_info {
    width: 332px;
    padding: 15px;
    background: #fff;
    .number-wrapper {
      display: flex;
      align-items: center;
      .price {
        color: #ff3838;
        font-size: 25px;

        span {
          font-size: 15px;
        }
      }
      .old-price {
        font-size: 15px;
        margin-left: 10px;
        color: #333333;
      }
    }
    .name {
      font-size: 16px;
      color: #333;
    }
    .msg {
      display: flex;
      align-items: center;
      justify-content: space-between;
      margin-top: 10px;
      .item {
        color: #999999;
        font-size: 14px;
      }
    }
  }
  .con-box {
    margin-top: 10px;
    padding-bottom: 20px;
    background: #f0f2f5;
    width: 335px;
    .content ::v-deepvideo {
      width: 100% !important;
    }
  }
}
</style>
