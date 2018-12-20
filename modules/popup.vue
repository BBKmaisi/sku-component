<template>
    <div class="popup" :class="{ 'popup--show': visible, 'popup_opacity': opacity }" :catchtouchmove="catchtouchmove">
        <div class="popup__mask" @click.stop="handleModalClick"></div>
            <div class="popup__container" :class="popupPositionClass" >
            <div v-if="showClose" class="popup-close-btn" @click.stop="close">
                <i class="iconfont xd-close"/>
            </div>
            <slot/>
        </div>
    </div>
</template>

<script>
    export default {
    props: {
        value: Boolean,
        transition: String,
        showClose: { //是否展示关闭按钮
            type: Boolean,
            default: false,
        },
        overlay: {
            type: Boolean,
            default: true
        },
        closeOnClickOverlay: {
            type: Boolean,
            default: true
        },
        position: {
            type: String,
            default: 'bottom'
        },
        catchtouchmove: {
            type: Boolean,
            default: false
        },
        closeCallback: {
            type: Function
        },
        opacity: Boolean, // 背景不透明
    },
    data() {
        return {
            visible: false
        }
    },
    watch: {
        value(val) {
            this.visible = val;
        },
        visible(val) {
            this.$emit('input', val);
        }
    },
    computed:{
        popupPositionClass(){
            return `popup-position--${this.position}`;
        }
    },

    methods: {
        handleModalClick(){
            if(this.closeOnClickOverlay){
                this.close();
            }
        },

        togglePopup() {
            this.visible = !this.visible
        },

        close(){
            this.visible = false;
            this.closeCallback&&this.closeCallback()
        }
    }
  }
</script>

<style lang="less" scoped>
.popup{
    visibility: hidden;
}
.popup.popup--show{
    visibility: visible;
}
.popup--show .popup__mask{
    opacity: 1;
    overflow: hidden;
}
.popup__bg_image{
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background-size: 100%;
    background-color:#ffffff;
    background-repeat: no-repeat;
}
.popup__mask {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    z-index: 1000;
    background-color: rgba(0, 0, 0, 0.5);
    opacity: 0;
    transition: all 0.5s ease;
}
.popup--show .popup__container{
    opacity: 1;
}
.popup__container {
    position: fixed;
    /*left: 50%;*/
    /*top: 50%;*/
    transform-origin: center;
    transition: all 0.3s ease;
    z-index: 1001;
    opacity: 0;
    & .popup-close-btn{
        position: absolute;
        top: -80px;
        right: 40px;
        border-radius: 27px;
        width: 54px;
        height: 54px;
        text-align: center;
        line-height: 54px;
        border:solid 1px #fff;
        color:#fff;
        font-size: 24px;
        &::after{
            content: "";
            border-left: solid 1px #fff;
            height: 26px;
            left: 26px;
            position: absolute;
        }
    }
}
.popup__container.popup-position--bottom{
    top: auto;
    left: 0;
    right: 0;
    bottom: 0;
    transform: translateY(100%);
}
.popup--show .popup__container.popup-position--bottom{
    transform: translateY(0);
}
.popup__container.popup-position--center{
    opacity: 0;
    left: 50%;
    top: 50%;
    transform: translate3d(-50%, -50%, 0);
}
.popup--show .popup__container.popup-position--center{
    opacity: 1;
}
.popup_opacity {
    .popup__mask {
        background-color: #000;
        transition: none;
    }
    .popup-close-btn{
        &::after{
            content: none!important;
        }
    }
}
</style>