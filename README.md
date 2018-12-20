## 基于mpvue的sku组件

### 基础用法
```html
<sku-select
    v-model="showBase"
    :sku="sku"
    :initialSku="initialSku"
    :goods="goods"
    :goods-id="goodsId"
    :hide-stock="sku.hide_stock"
    :quota="quota"
    :quota-used="quotaUsed"
    :skuSubmitType="skuSubmitType"
    :resetStepperOnHide="resetStepperOnHide"
    :resetSelectedSkuOnHide="resetSelectedSkuOnHide"
    @submit="skuSubmit"
    @sku-close="skuSelectCallBack"
/>
```


### API

| 参数 | 说明 | 类型 | 默认值 |
|-----------|-----------|-----------|-------------|
| v-model | 是否显示sku | `Boolean` | `false` |
| sku | 商品sku数据 | `Object` | - |
| goods | 商品信息 | `Object` | - |
| goods-id | 商品id | `String | Number` | - |
| hide-stock | 是否显示商品剩余库存 | `Boolean` | `false` |
| show-add-cart-btn | 是否显示加入购物车按钮 | `Boolean` | `true` |
| quota | 限购数(0表示不限购) | `Number` | `0` |
| quota-used | 已经购买过的数量 | `Number` | `0` |
| reset-stepper-on-hide | 窗口隐藏时重置选择的商品数量 | `Boolean` | `false` |
| reset-selected-sku-on-hide | 窗口隐藏时重置已选择的sku | `Boolean` | `false` |
| disable-stepper-input | 是否禁用sku中stepper的input框 | `Boolean` | `false` |
| close-on-click-overlay | 点击popup的overlay后是否关闭弹窗 | `Boolean` | `false` |

### Event

| 事件名 | 说明 | 参数 |
|-----------|-----------|-----------|
| submit | 点击购买回调 | skuData: Object |
| stepper-change | 购买数量变化时触发 | value: number |

### 数据结构
#### sku对象结构
```javascript
sku: {
  // 所有sku规格类目与其值的从属关系，比如商品有颜色和尺码两大类规格，颜色下面又有红色和蓝色两个规格值。
  // 可以理解为一个商品可以有多个规格类目，一个规格类目下可以有多个规格值。
  tree: [
    {
      k: '颜色', // skuKeyName：规格类目名称
      v: [
        {
          id: '30349', // skuValueId：规格值 id
          name: '红色', // skuValueName：规格值名称
          imgUrl: 'https://img.yzcdn.cn/1.jpg' // 规格类目图片，只有第一个规格类目可以定义图片
        },
        {
          id: '1215',
          name: '蓝色',
          imgUrl: 'https://img.yzcdn.cn/2.jpg'
        }
      ],
      k_s: 's1' // skuKeyStr：sku 组合列表（下方 list）中当前类目对应的 key 值，value 值会是从属于当前类目的一个规格值 id
    }
  ],
  // 所有 sku 的组合列表，比如红色、M 码为一个 sku 组合，红色、S 码为另一个组合
  list: [
    {
      id: 2259, // skuId，下单时后端需要
      price: 100, // 价格（单位分）
      s1: '1215', // 规格类目 k_s 为 s1 的对应规格值 id
      s2: '1193', // 规格类目 k_s 为 s2 的对应规格值 id
      s3: '0', // 最多包含3个规格值，为0表示不存在该规格
      stock_num: 110 // 当前 sku 组合对应的库存
    }
  ],
  price: '1.00', // 默认价格（单位元）
  stock_num: 227, // 商品总库存
  collection_id: 2261, // 无规格商品 skuId 取 collection_id，否则取所选 sku 组合对应的 id
  none_sku: false, // 是否无规格商品
  hide_stock: false // 是否隐藏剩余库存
}
```

#### goods 对象结构
```javascript
goods: {
  // 商品标题
  title: '测试商品',
  // 默认商品 sku 缩略图
  picture: 'https://img.yzcdn.cn/1.jpg'
}
```

### 处理

使用的时候，可能后端数据返回和组件需要的数据不同，可以自己处理一下后端数据。

