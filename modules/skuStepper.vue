<template>
    <div class="sku-stepper">
        <div class="container flex f-jc-sb f-ai-c">
            <div class="title">{{ stepperTitle || '购买数量' }}：</div>
            <div class="stepper">
            <stepper
                v-model="currentNum"
                :min="1"
                :max="stepperLimit"
                :disable-input="disableStepperInput"
                @overlimit="onOverLimit"
                @change="onChange"
            />
            </div>
        </div>
        <div v-if="!hideStock" class="stock">剩余{{ stock }}件</div>
        <div v-if="quotaText" class="quota">{{ quotaText }}</div>
    </div>
</template>

<script>
import Stepper from '@/components/stepper';
import { LIMIT_TYPE } from '../constants.js';
import { eventBus } from '../eventbus.js';
const { QUOTA_LIMIT, STOCK_LIMIT } = LIMIT_TYPE;

export default {
    name: 'sku-stepper',
    components: {
        Stepper
    },
    props: {
        skuStockNum: Number,
        selectedSku: Object,
        selectedSkuComb: Object,
        selectedNum: Number,
        stepperTitle: String,
        quota: Number,
        quotaUsed: Number,
        hideStock: Boolean,
        disableStepperInput: Boolean,
        customStepperConfig: Object
    },
    data() {
        return {
            currentNum: this.selectedNum,
            // 购买限制类型: 限购/库存
            limitType: STOCK_LIMIT
        };
    },

    watch: {
        currentNum(num) {
	        eventBus.$emit('sku-numChange', num);
        },

        stepperLimit(limit) {
            if (limit < this.currentNum) {
                this.currentNum = limit;
            }
        }
    },

    computed: {
        stock() {
            if (this.selectedSkuComb) {
                return this.selectedSkuComb.stock_num;
            }
            return this.skuStockNum;
        },
        quotaText() {
            const { quotaText } = this.customStepperConfig;
            let text = '';
            if (quotaText) {
                text = quotaText;
            } else if (this.quota > 0) {
                text = `限购${this.quota}件`;
            }
            return text;
        },

        stepperLimit() {
            // const quotaLimit = this.quota - this.quotaUsed;
            let limit;
            // 无限购时直接取库存，有限购时取限购数和库存数中小的那个
            if (this.quota > 0 && this.quota <= this.stock) {
                // 修正负的limit
                limit = this.quota;
                this.limitType = QUOTA_LIMIT;
            } else {
                limit = this.stock;
                this.limitType = STOCK_LIMIT;
            }
            return limit;
        }
    },

    methods: {
        setCurrentNum(num) {
            this.currentNum = num;
        },

        onOverLimit(action) {
	        eventBus.$emit('sku-overLimit', {
                action,
                limitType: this.limitType,
                quota: this.quota,
                quotaUsed: this.quotaUsed
            });
        },

        onChange(currentValue) {
            const { handleStepperChange } = this.customStepperConfig;
            handleStepperChange && handleStepperChange(currentValue);
            this.$emit('change', currentValue);
        }
    }
}
</script>

<style lang="less" scoped>
.sku-stepper{
    padding: 20px 40px;
    border-top: 2px solid #fafafa;
}
.container{
    padding: 5px 0;
}
.title{
    font-size: 28px;
    color: #333333;
}
.stock{
    display: inline-block;
    margin-right: 20px;
    color: #9e9e9e;
    font-size: 24px;
}
.quota {
    display: inline-block;
    color: #ff7722;
    font-size: 24px;
}
</style>