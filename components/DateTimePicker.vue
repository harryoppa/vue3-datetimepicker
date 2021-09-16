<template>
    <div class="tvh-dateTimePickerWrapper" ref="wrapper">
        <div class="input-group input-search" @click="openHandler" :class="{'input-group-lg': size === 'lg'}">
            <input type="text"
                   :value="selectDateString"
                   class="form-control m-0"
                   :class="[inputClass, {'form-control-lg': size === 'lg'}]"
                   :placeholder="placeholder" aria-label="Username" aria-describedby="basic-addon1">
            <span class="input-group-text" id="basic-addon1">
                <svg xmlns="http://www.w3.org/2000/svg" class="ionicon" viewBox="0 0 512 512"><title>Calendar</title><rect fill="none" stroke="currentColor" stroke-linejoin="round" stroke-width="32" x="48" y="80" width="416" height="384" rx="48"/><circle cx="296" cy="232" r="24"/><circle cx="376" cy="232" r="24"/><circle cx="296" cy="312" r="24"/><circle cx="376" cy="312" r="24"/><circle cx="136" cy="312" r="24"/><circle cx="216" cy="312" r="24"/><circle cx="136" cy="392" r="24"/><circle cx="216" cy="392" r="24"/><circle cx="296" cy="392" r="24"/><path fill="none" stroke="currentColor" stroke-linejoin="round" stroke-width="32" stroke-linecap="round" d="M128 48v32M384 48v32"/><path fill="none" stroke="currentColor" stroke-linejoin="round" stroke-width="32" d="M464 160H48"/></svg>
            </span>
        </div>

        <date-time-picker-modal
            v-if="isOpen"
            :class="{ 'tvh-fadeInDown': isOpen }"
            :singleDate="singleDate"
            :startDate="startDate"
            :endDate="endDate"
            :timeFormat="timeFormat"
            @submitHandler="submitHandler"
            @cancelHandler="isOpen = false"
            :style="{
        marginLeft: `-${shiftMarginLeft}px`,
        marginTop: `-${shiftMarginHeight}px`
      }"
        />
    </div>
</template>

<script>
import DateTimePickerModal from "./DateTimePickerModal.vue";
import * as utils from "../lib/date";
import { getTimeObjectFromDate } from "../lib/time";

const BOX_LENGTH = 750; //px
const BOX_HEIGHT = 510; //px
const RWD_THRESHOLD_W = 700;

const _getDateString = (date, format = "hh:mm:A") => {
    if (!date) return "";

    const startYear = date.getFullYear();
    const startMonth = utils.monthShortConfig[date.getMonth()];
    const starDate = date.getDate();

    const timeObject = getTimeObjectFromDate(date, format);
    const hh = timeObject.hh;
    const HH = timeObject.HH;
    const mm = timeObject.mm;
    const a = timeObject.A;

    if (HH) return `${startYear} ${startMonth} ${starDate}  ${HH}:${mm}`;

    return `${startYear} ${startMonth} ${starDate}  ${hh}:${mm} ${a}`;
};

export default {
    name: "DateTimePicker",
    components: { DateTimePickerModal },
    props: {
        startDate: Date,
        endDate: Date,
        minDate: Date,
        timeFormat: {
            type: String,
            default: "hh:mm:A"
        },
        singleDate: {
            type: Boolean,
            default: false
        },
        onChange: Function,
        inputClass: String,
        placeholder: String,
        size: {
            type: String,
            default: 'lg'
        }
    },
    methods: {
        calculateShift: function() {
            const windowWidth = window.innerWidth;
            const windowHeight = window.innerHeight;

            const wrapper = this.$refs.wrapper;

            const { x, y } = wrapper.getBoundingClientRect();

            const dx = windowWidth - x;
            const isDesktop = windowWidth > RWD_THRESHOLD_W;

            // calculate shift x
            if (dx < BOX_LENGTH && isDesktop) {
                this.shiftMarginLeft = Math.min(BOX_LENGTH - dx, x);
            }

            // calculate shift y, has enough space
            if (y > windowHeight / 2 && windowHeight > 2 * BOX_HEIGHT && isDesktop) {
                this.shiftMarginHeight = BOX_HEIGHT;
            }

            // calculate shift y, has no enough space
            if (windowHeight < 2 * BOX_HEIGHT && isDesktop) {
                const dy = windowHeight - y;
                this.shiftMarginHeight = Math.min(BOX_HEIGHT - dy, y);
            }
        },
        openHandler: function() {
            this.calculateShift();
            return (this.isOpen = !this.isOpen);
        },
        getDateString: function(data) {
            const { singleDate, timeFormat } = this;
            const { startDate, endDate } = data;
            return singleDate
                ? _getDateString(startDate, timeFormat)
                : `${_getDateString(startDate, timeFormat)} - ${_getDateString(
                    endDate,
                    timeFormat
                )}`;
        },
        callOnChange: function(returnData) {
            return this.$emit("onChange", { ...returnData });
        },
        submitHandler: function(data) {
            this.isOpen = false;
            this.selectDateString = this.getDateString(data);
            this.$emit('update:date', data);
            return this.callOnChange(data);
        },
        closeHandler(evt) {
            if (!document.querySelector('.tvh-dateTimeWrapper').contains(evt.target))
            this.isOpen = false;
        }
    },
    data() {
        const { startDate, endDate } = this;
        return {
            isOpen: false,
            shiftMarginLeft: 0,
            shiftMarginHeight: 0,
            selectDateString: !startDate
                ? ""
                : this.getDateString({
                    startDate,
                    endDate
                })
        };
    },
    watch: {
        isOpen() {
            if (this.isOpen) {
                setTimeout(() => {
                    document.addEventListener('click', this.closeHandler)
                }, 100)
            } else {
                document.removeEventListener('click', this.closeHandler)
            }
        },
        minDate() {
            this.selectDateString = _getDateString(this.minDate, this.timeFormat);
        }
    },
    provide() {
        return {
            startDate: this.startDate,
            endDate: this.endDate,
            minDate: () => this.minDate,
            onChange: this.onChange,
        }
    }
};
</script>

<style lang="scss" scoped>
@import "../style/main";

/* animation */
@keyframes fadeInDown {
    0% {
        opacity: 0;
        transform: translateY(-20px);
    }
    100% {
        opacity: 1;
        transform: translateY(0);
    }
}

.tvh-fadeInDown {
    animation: fadeInDown 0.6s ease both;
}

/* compomnent style */
.tvh-dateTimePickerWrapper {
    position: relative;

    //modal
    .tvh-dateTimeWrapper {
        opacity: 0;
        left: 0;
        position: absolute;
        z-index: 198;
    }

    .input-search {
        .input-group-text {
            svg {
                width: 24px;
                height: 24px;
                margin-top: 2px;
                margin-left: -2px;
            }
        }
    }
}
</style>
