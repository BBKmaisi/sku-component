<template>
    <div class="sku-header">
        <div class="img-wrap">
            <img :src="goodsImg" class="image-item">
        </div>
        <div class="goods-info">
            <div class="goods-name elip-hidden" style="-webkit-line-clamp: 2">{{ goods.title }}</div>
            <div class="goods-price flex f-ai-c f-ff-rn">
                <div class="price-symbol iconfont xd-rmb"></div>
                <div class="price-num">{{ price }}</div>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    name: 'sku-header',

    props: {
        sku: Object,
        goods: Object,
        selectedSku: Object,
        price: String
    },

    computed: {
        goodsImg() {
            const s1Id = this.selectedSku.styleId;
            const skuImg = this.getSkuImg(s1Id);
            // 优先使用选中sku的图片
            return skuImg || this.goods.picture;
        }
    },

    methods: {
        getSkuImg(id) {
            if (!id) return;
            // 目前skuImg都挂载在skuTree中style那类sku上
            const treeItem = this.sku.tree.filter(treeItem => treeItem.k_s === 'styleId')[0] || {};
            if (!treeItem.v) {
                return;
            }
            const matchedSku = treeItem.v.filter(skuValue => skuValue.id === id)[0];
            if (matchedSku && matchedSku.imgUrl) {
                return matchedSku.imgUrl;
            }
        }
    }
}
</script>

<style lang="less" scoped>
.sku-header{
    height: 210px;
    margin-left: 40px;
    padding-bottom: 20px;
    border-bottom: 2px solid #fafafa;
}
.img-wrap{
    position: relative;
    float: left;
    margin-top: -30px;
    width: 210px;
    height: 210px;
    background: #fafafa;
    border-radius: 10px;
    .image-item{
        position: absolute;
        margin: auto;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        max-width: 100%;
        max-height: 100%;
        border-radius: 10px;
    }
}
.goods-info{
    padding: 20px 100px 10px 20px;
    min-height: 120px;
    overflow: hidden;
    box-sizing: border-box;
}
.goods-name{
    height: 64px;
    line-height: 32px;
    padding-left: 4px;
    font-size: 26px;
}
.goods-price{
    margin-top: 8px;
    color: #f44336;
    vertical-align: middle;
}
.price-symbol{
    margin-top: 4px;
    font-size: 22px;
}
.price-num{
    font-size: 26px;
    font-weight: 500;
}
.close-icon {
    width: 80px;
    height: 100px;
    top: 10px;
    right: 20px;
    font-size: 40px;
    color: #9e9e9e;
    position: absolute;
    text-align: center;
}
</style>
