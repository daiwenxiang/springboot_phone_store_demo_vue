<template>
  <div>

    <van-row>
      <van-col span="24">
        <van-tabs :line-height="2" :line-width="100" @click="onClick" color="#E32DAB" sticky
                  title-active-color="#E32DAB">

          <van-tab :title="categories[index-1].name" class="tab" v-for="index in categories.length">

            <van-card :desc="item.desc"
                      :price="item.price"
                      :thumb="item.thumb"
                      :title="item.title"
                      v-for="(item,index) in phones"
            >
              <template #tags>
                <van-tag color="#f2826a" style="margin-left: 5px;" v-for="tag in item.tag">{{tag.name}}</van-tag>
              </template>
              <template #footer>
                <van-button @click="buy(index)" round size="mini" type="info">购买</van-button>
              </template>
            </van-card>

          </van-tab>
        </van-tabs>
      </van-col>
    </van-row>

    <van-sku
        :goods="goods"
        :hide-stock="sku.hide_stock"
        :sku="sku"
        @buy-clicked="onBuyClicked"
        v-model="show"
    >
      <template #sku-actions="props">
        <div class="van-sku-actions">

          <!-- 直接触发 sku 内部事件，通过内部事件执行 onBuyClicked 回调 -->
          <van-button
              @click="props.skuEventBus.$emit('sku:buy')"
              size="large"
              square
              type="danger"
          >
            买买买
          </van-button>
        </div>
      </template>
    </van-sku>

  </div>


</template>

<script>
  import {PullRefresh, Swipe, SwipeItem} from 'vant'

  export default {
    comments: {
      [PullRefresh.name]: PullRefresh,
      [Swipe.name]: Swipe,
      [SwipeItem.name]: SwipeItem
    },
    data() {
      return {
        categories: '',
        phones: '',
        show: true,
        sku: {},
        goods: {}
      }
    },
    created() {
      const _this = this
      axios.get('http://localhost:8080/phone/index').then(function (resp) {
        _this.phones = resp.data.data.phones
        _this.categories = resp.data.data.categories
      })
    },
    methods: {
      onClick(index) {
        const _this = this
        axios.get('http://localhost:8080/phone/findByCategoryType/' + this.categories[index].type).then(function (resp) {
          _this.phones = resp.data.data
        })
      },
      buy(index) {
        this.show = true
        const _this = this
        axios.get('http://localhost:8080/phone/findSpecsByPhoneId/' + this.phones[index].id).then(function (resp) {
          _this.goods = resp.data.data.goods
          _this.sku = resp.data.data.sku
        })
      },
      onBuyClicked(item) {
        this.$store.state.specsId = item.selectedSkuComb.s1
        this.$store.state.quantity = item.selectedNum
        this.$router.push('/addressList')
      }
    }
  }
</script>