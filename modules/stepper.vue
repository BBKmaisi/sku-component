<template>
    <div class="c-stepper flex">
        <button
            class="minus flex f-ai-c f-jc-c"
            :class="{ disabled: minusDisabled }"
            @click="onChange('minus')"
        >
            <div class="icon iconfont xd-minus"></div>
        </button>
        <input
            type="number"
            class="input flex f-ai-c f-jc-c"
            :value="currentValue"
            :disabled="disabled || disableInput"
            @input="onInput"
            @blur="onBlur"
        >
        <button
            class="plus flex f-ai-c f-jc-c"
            :class="{ disabled: plusDisabled, isLive: isLive && !plusDisabled }"
            @click="onChange('plus')"
        >
            <div class="icon iconfont xd-add"></div>
        </button>
    </div>
</template>

<script>

export default {
    name: 'stepper',
    props: {
        value: {},
        integer: Boolean,
        disabled: Boolean,
        disableInput: Boolean,
        min: {
            type: [String, Number],
            default: 1
        },
        max: {
            type: [String, Number],
            default: Infinity
        },
        step: {
            type: [String, Number],
            default: 1
        },
        defaultValue: {
            type: [String, Number],
            default: 1
		},
		styleType: String,
    },
    data() {
        let value = this.value ? +this.value : +this.defaultValue;
        const correctedValue = this.correctValue(value);
        if (value !== correctedValue) {
            value = correctedValue;
            this.$emit('input', value);
        }
        return {
            currentValue: value
        };
    },

    computed: {
        minusDisabled() {
            const min = +this.min;
            const step = +this.step;
            const currentValue = +this.currentValue;
            return min === currentValue || (currentValue - step) < min || this.disabled;
        },
        plusDisabled() {
            const max = +this.max;
            const step = +this.step;
            const currentValue = +this.currentValue;
            return max === currentValue || (currentValue + step) > max || this.disabled;
		},
		isLive(){
			return this.styleType == 'live';
		}
    },
    watch: {
        value(val) {
            if (val !== '') {
                val = this.correctValue(+val);
                if (val !== this.currentValue) {
                    this.currentValue = val;
                }
            }
        }
    },
    methods: {
        correctValue(value) {
            if (isNaN(value)) {
                return this.min;
            }
            value = Math.max(this.min, Math.min(this.max, value));
            return this.integer ? Math.floor(value) : value;
        },
        onInput(event) {
            const { value } = event.target;
            this.currentValue = value ? this.correctValue(+value) : value;
            event.target.value = this.currentValue;
            this.emitInput();
        },
        onChange(type) {
            if ((this.minusDisabled && type === 'minus') || (this.plusDisabled && type === 'plus')) {
                this.$emit('overlimit', type);
                return;
            }
            const step = +this.step;
            const currentValue = +this.currentValue;
            this.currentValue = type === 'minus' ? (currentValue - step) : (currentValue + step);
            this.emitInput();
            this.$emit(type);
        },
        onBlur(event) {
            if (!this.value) {
                this.currentValue = +this.min;
                this.emitInput();
            }
            this.$emit('blur', event);
        },
        emitInput() {
            this.$emit('input', this.currentValue);
            this.$emit('change', this.currentValue);
        }
    }
}
</script>

<style lang="less" scoped>
.c-stepper{
    font-size: 0;
    input.input {
        width: 70px;
		height: 60px;
		padding: 2px;
		padding-top: 12px;
		line-height: 60px;
        border: 2px solid #e5e5e5;
        box-sizing: border-box;
        color: #333;
        font-size: 24px;
        vertical-align: middle;
        text-align: center;
        -webkit-appearance: none;
        &[disabled] {
            color: #333;
        }
    }
}
.minus,.plus{
    width: 70px;
    height: 60px;
    box-sizing: border-box;
    background-color: #ffffff;
    border: 2px solid #e5e5e5;
    position: relative;
    padding: 5px;
    vertical-align: middle;
    color: #8e8e8e;
    &:active {
        background-color: #f8f8f8;
    }
}
.icon{
    font-size: 20px;
}
.minus {
    border-radius: 4px 0 0 4px;
    border-right: none;
}
.plus {
    border-radius: 0 4px 4px 0;
	border-left: none;
	&.isLive {
		color: #ff5777;
	}
}
.disabled {
    background-color: #f8f8f8;
    &:active {
        background-color: #f8f8f8;
    }
}
input[type="number"]::-webkit-inner-spin-button,
input[type="number"]::-webkit-outer-spin-button {
    -webkit-appearance: none;
    margin: 0;
}
</style>
