<template>
    <div class="sku-actions flex f-ai-c f-jc-c">
        <button class="submit flex f-ai-c f-jc-c" @click="submit">{{ buyText }}</button>
    </div>
</template>

<script>
import { eventBus } from '../eventbus';
export default {
    name: 'sku-actions',
    props: {
        buyText: {
            type: String,
            default: "确认"
        },
        showAddCartBtn: Boolean
    },
    computed: {},
    methods: {
        submit(){
	        eventBus.$emit('sku-submit')
        }
    }
}
</script>

<style lang="less" scoped>
.sku-actions{
    width: 100%;
    height: 120px;
    margin-top: 20px;
    border-top: 2px solid #fafafa;
}
.submit{
    width: 670px;
    height: 80px;
    font-size: 28px;
    font-weight: 500;
    border: none;
    border-radius: 40px;
    color: #ffffff;
    background-image: linear-gradient(256deg, #FF5622, #FFB075);
}
</style>