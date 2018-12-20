<template>
	<div class="sku-component">
        <popup v-model="show" :showClose="true" v-if="!isSkuEmpty">
            <div class="sku-content">
                <!-- 头部 -->
                <sku-header
                    :selected-sku="selectedSku"
                    :goods="goods"
                    :sku="sku"
                    :price="price"
                >
                </sku-header>
                <div class="sku-body">
                    <!-- sku选择 -->
                    <div v-if="hasSku" class="sku-group-container">
                        <div
                            v-for="(skuTreeItem, rowIndex) in skuTree"
                            :key="rowIndex"
                        >
                            <div class="title">{{ skuTreeItem.k }}：</div>
                            <div class="flex f-fw-w">
                                <!-- sku-row-item -->
                                <div
                                    class="sku-row-item flex f-ai-c f-jc-c"
                                    :key="itemIndex"
                                    v-for="(skuValue, itemIndex) in skuTreeItem.v"
                                    :class="{ active: skuValue.id === selectedSku[skuTreeItem.k_s], disabled: !isChoosable[skuValue.id] }"
                                    :style=" skuValue.id === selectedSku[skuTreeItem.k_s] ? themeItemStyle : '' "
                                    @click="onSelect({
                                        id: skuValue.id,
                                        skuKeyStr: skuTreeItem.k_s
                                    })"
                                >
                                    {{ skuValue.name }}
                                </div>
                            </div>
                        </div>
                    </div>
                    <div v-else class="sku-group-container">
                        <div class="title">单品</div>
                        <div class="flex f-fw-w">
                            <div class="sku-row-item flex f-ai-c f-jc-c active" :style="themeItemStyle">
                                默认
                            </div>
                        </div>
                    </div>
                    <!-- 数目 -->
                    <sku-stepper
                        ref="skuStepper"
                        :selected-sku="selectedSku"
                        :selected-sku-comb="selectedSkuComb"
                        :selected-num="selectedNum"
                        :stepper-title="stepperTitle"
                        :sku-stock-num="sku.stock_num"
                        :quota="quota"
                        :quota-used="quotaUsed"
                        :disable-stepper-input="disableStepperInput"
                        :hide-stock="hideStock"
                        :custom-stepper-config="customStepperConfig"
                        @change="$emit('stepper-change', $event)"
                    />
                </div>
                <!-- 按钮 -->
                <sku-actions
                    :buy-text="buyText"
                    :show-add-cart-btn="showAddCartBtn"
                />
            </div>
        </popup>
    </div>
</template>

<script>
import { eventBus } from './eventbus';
import popup from './modules/popup/index'
import skuHeader from './modules/skuHeader'
import skuStepper from './modules/skuStepper'
import skuActions from './modules/skuActions'
import {
    isAllSelected,
    isSkuChoosable,
    getSkuComb,
    getSelectedSkuValues
} from './skuHelper.js'
import { LIMIT_TYPE, UNSELECTED_SKU_VALUE_ID } from './constants.js'

const { QUOTA_LIMIT } = LIMIT_TYPE;
export default {
    name: 'sku-component',
    components:{
        popup,
        skuHeader,
        skuStepper,
        skuActions
    },
	props: {
        sku: Object,
        goods: Object,
        value: Boolean,
        buyText: String,
        goodsId: [Number, String],
        stepperTitle: String,
        hideStock: Boolean,
        skuSubmitType: String,
        resetStepperOnHide: Boolean,
        resetSelectedSkuOnHide: Boolean,
        disableStepperInput: Boolean,
        closeOnClickOverlay: Boolean,
        initialSku: {
            type: Object,
            default: () => ({})
        },
        quota: {
            type: Number,
            default: 0
        },
        quotaUsed: {
            type: Number,
            default: 0
        },
        showAddCartBtn: {
            type: Boolean,
            default: true
        },
        customStepperConfig: {
            type: Object,
            default: () => ({})
        }
    },

    data() {
        return {
            selectedSku: {},
            selectedNum: 1,
            show: this.value,
            isChoosable: {}
        };
    },

    watch: {
        show(val) {
            // 触发外面的v-model
            this.$emit('input', val);
            if(!val){
                const selectedSkuValues = getSelectedSkuValues(
                    this.sku.tree,
                    this.selectedSku
                );
                this.$emit('sku-close', {
                    selectedSkuValues,
                    selectedNum: this.selectedNum,
                    selectedSkuComb: this.selectedSkuComb
                });
                if (this.resetStepperOnHide) {
                    this.$refs.skuStepper && this.$refs.skuStepper.setCurrentNum(1);
                }
                if (this.resetSelectedSkuOnHide) {
                    this.resetSelectedSku(this.skuTree);
                }
            }
        },
        value(val) {
            this.show = val;
        },
        skuTree(val) {
            this.resetSelectedSku(val);
        }
    },

    computed: {
        isSkuEmpty() {
            return Object.keys(this.sku).length === 0;
        },
        hasSku() {
            return !this.sku.none_sku;
        },
        isSkuCombSelected() {
        	return isAllSelected(this.sku.tree, this.selectedSku);
        },
        selectedSkuComb() {
            if (!this.hasSku) {
                return {
                    id: this.sku.collection_id,
                    price: this.sku.price,
                    stock_num: this.sku.stock_num
                };
            } else if (this.isSkuCombSelected) {
                return getSkuComb(this.sku.list, this.selectedSku);
            }
            return null;
        },
        price() {
            if (this.selectedSkuComb) {
                return this.selectedSkuComb.price;
            }
            return this.sku.price;
        },
        skuTree() {
            return this.sku.tree || [];
        },
        themeItemStyle(){
            // 可以根据需求自定义
            return 'background-color: #fa3968; border-color: #fa3968'
        }
    },

    mounted() {
        // 定义监听函数
	    eventBus.$on('sku-close', this.onClose)
	    eventBus.$on('sku-numChange', this.onNumChange);
	    eventBus.$on('sku-overLimit', this.onOverLimit);
	    eventBus.$on('sku-submit', this.onSubmit);
        this.resetSelectedSku(this.skuTree);
    },
	destroyed(){
		eventBus.$off('sku-close');
		eventBus.$off('sku-numChange');
		eventBus.$off('sku-overLimit');
		eventBus.$off('sku-submit');
    },
	methods: {
        /**
         *  关闭popup，监听
         */
        onClose() {
            this.show = false;
        },
        /**
         *  重置sku
         */
        resetSelectedSku(skuTree) {
            let selectedSku = {}
            // 重置selectedSku
            skuTree.forEach(item => {
                selectedSku[item.k_s] = this.initialSku&&this.initialSku[item.k_s] || UNSELECTED_SKU_VALUE_ID;
            });
            // 只有一个sku规格值时默认选中
            skuTree.forEach(item => {
                const key = item.k_s;
                const valueId = item.v[0].id;
                if (
                    item.v.length === 1 &&
                    isSkuChoosable(this.sku.list, this.selectedSku, { key, valueId })
                ) {
                    selectedSku[key] = valueId;
                }
            });
            this.selectedSku = selectedSku
            // 设置是否可选
            this.setChoosable()
        },
        /**
         *  选择sku，监听
         */
        onSelect(skuValue) {
            if(this.isChoosable[skuValue.id]){
                // 点击已选中的sku时则取消选中
                this.selectedSku =
                    this.selectedSku[skuValue.skuKeyStr] === skuValue.id
                    ? { ...this.selectedSku, [skuValue.skuKeyStr]: UNSELECTED_SKU_VALUE_ID }
                    : { ...this.selectedSku, [skuValue.skuKeyStr]: skuValue.id };
                // 设置是否可选
                this.setChoosable()
                // 如果外侧有sku-select，可以触发
                this.$emit('sku-selected', {
                    skuValue,
                    selectedSku: this.selectedSku,
                    selectedSkuComb: this.selectedSkuComb
                });
            }
        },
        /**
         *  设置是否可选
         */
        setChoosable(){
            let obj = {}
            this.skuTree&&this.skuTree.map((skuTreeItem)=>{
                skuTreeItem&&skuTreeItem.v&&skuTreeItem.v.map((skuValue)=>{
                    let isChoosable = this.isChoosableCallBack(skuTreeItem.k_s,skuValue.id)
                    obj = Object.assign({},obj,{
                        [skuValue.id]: isChoosable
                    })
                })
            })
            this.isChoosable = obj
        },
        /**
         *  判断是否可选
         */
        isChoosableCallBack(skuKeyStr,id) {
            return isSkuChoosable(this.sku.list, this.selectedSku, {
                key: skuKeyStr,
                valueId: id
            });
        },
        /**
         *  修改数目，监听
         */
        onNumChange(num) {
            this.selectedNum = num;
        },
        /**
         *  限购，监听
         */
        onOverLimit(data) {
            const { action, limitType, quota, quotaUsed } = data;
            const { handleOverLimit } = this.customStepperConfig;
            if (handleOverLimit) {
                handleOverLimit(data);
                return;
            }
            if (action === 'minus') {
                this.$toast({ title: "请至少选择一件" });
            } else if (action === 'plus') {
                if (limitType === QUOTA_LIMIT) {
                    let msg = `每人限购${quota}件`;
                    if (quotaUsed > 0) {msg += `，你已购买${quotaUsed}件`;}
                    this.$toast({ title: msg });
                } else {
                    this.$toast({ title: "库存不足" });
                }
            }
        },
        /**
         *  加购，监听
         */
        onSubmit() {
            const error = this.validateSku()
            if (error) {
                this.$toast({ title: error })
            } else {
                this.onClose()
                this.$emit("submit", this.getSkuData())
            }
        },
        validateSku() {
            if (this.selectedNum === 0) {
                return '商品已经无法购买啦';
            }
            // 达到限购数
            if (this.quota && this.quotaUsed && this.quota <= this.quotaUsed) {
                return `每人限购${this.quota}件，你已购买${this.quotaUsed}件`;
            }
            if (this.isSkuCombSelected) {
                return "";
            }
            return '请先选择商品规格';
        },
        /**
         *  获取sku选择
         */
        getSkuData() {
            return {
                itemId: this.goodsId,
                selectedNum: this.selectedNum,
                skuId: this.selectedSkuComb.id,
                selectedSkuComb: this.selectedSkuComb
            };
        }
    }
}
</script>

<style lang="less" scoped>
.sku-content{
    width: 750px;
    background: #ffffff;
    .sku-body{
        max-height: 680px;
        overflow-y: scroll;
        -webkit-overflow-scrolling: touch;
        ::-webkit-scrollbar {
            display: none;
        }
    }
    .sku-group-container{
        padding: 20px 40px;
    }
    .title {
        padding-bottom: 20px;
        font-size: 28px;
        color: #333333;
    }
}
.sku-row-item{
    padding: 0 20px;
    margin: 0 25px 25px 0;
    height: 60px;
    min-width: 50px;
    font-size: 24px;
    color: #333333;
    background-color: #f8f8f8;
    border-radius: 8px;
}
.sku-row-item.active {
    color: #ffffff;
    border-color: #F8662A;
    background-color: #F8662A;
}
.sku-row-item.disabled {
    color: #9e9e9e;
}
</style>
