<template>
  <div>
    <ul class="tvh-calendar">
      <template v-for="(weekday, key) in weekdays" :key="'weekday' + key">
        <li class="tvh-weekday">
          <span>{{ weekday }}</span>
        </li>
      </template>
      <template v-for="(day, key) in startWeekday" :key="'null' + key">
        <li class="tvh-day">
          <!-- <span class="nullBlock"></span> -->
        </li>
      </template>
      <template v-for="(day, key) in daysCount" :key="'day' + key">
        <li class="tvh-day">
          <span
            v-if="!singleDate"
            :class="getDayStyle(day)"
            @click="updateSelectingDay(day)"
          >
            {{ day }}
          </span>

          <span
            v-if="singleDate"
            :class="getDayStyle(day)"
            @click="updateSelectingSingleDay(day)"
          >
            {{ day }}
          </span>
        </li>
      </template>
    </ul>
  </div>
</template>

<script>
import * as utils from "../../lib/date";

const isToday = otherDay => {
  const today = new Date();
  return utils.isSameDay(today, otherDay);
};

const isBetweenDays = (smallDay, bigDay, currentDay) => {
  if (currentDay < bigDay && smallDay < currentDay) return true;
  return false;
};

export default {
  name: "Calender",
  props: {
    year: Number,
    month: Number,
    startDate: Date,
    endDate: Date,
    onChange: Function,
    singleDate: {
      type: Boolean,
      default: false
    }
  },
  inject: ['minDate'],
  methods: {
    callOnChange: function(returnData) {
      return this.$emit("onChange", { ...returnData });
    },
    updateSelectingSingleDay: function(day) {
      if (!day) return;

      const { year, month, innerStartDate, innerEndDate } = this;
      const currentDay = new Date(`${year}-${month + 1}-${day}`);

      this.innerStartDate = currentDay;
      this.innerEndDate = currentDay;
      this.selectedDay = day;
      return this.callOnChange({
        selectedDay: currentDay,
        startDate: this.innerStartDate
      });
    },
    updateSelectingDay: function(day) {
      if (!day) return;

      const {
        year,
        month,
        innerStartDate,
        innerEndDate,
        isSelectingStartDay
      } = this;

      const currentDay = new Date(`${year}-${month + 1}-${day}`);

      // reset
      if (
        isSelectingStartDay ||
        (!isSelectingStartDay && currentDay < innerStartDate)
      ) {
        this.innerStartDate = currentDay;
        this.isSelectingStartDay = false;
      } else {
        this.isSelectingStartDay = true;
      }

      this.innerEndDate = currentDay;
      this.selectedDay = day;

      return this.callOnChange({
        selectedDay: currentDay,
        startDate: this.innerStartDate,
        endDate: this.innerEndDate
      });
    },

    getDayStyle: function(day) {
      const { innerStartDate, innerEndDate, year, month } = this;
      const currentDay = new Date(`${year}-${month + 1}-${day}`);
      const minDate = this.minDate();

      if (minDate && utils.isLessThan(currentDay, minDate)) return "tvh-disabled";
      if (utils.isSameDay(currentDay, innerStartDate)) return "tvh-innerStartDate";
      if (utils.isSameDay(currentDay, innerEndDate)) return "tvh-innerEndDate";
      if (isBetweenDays(innerStartDate, innerEndDate, currentDay))
        return "tvh-between";
      if (isToday(currentDay)) return "tvh-today";

      return "";
    }
  },
  computed: {
    startWeekday: function() {
      return utils.getWeekday(
        new Date(`${utils.lessThanTen(this.year)}-${utils.lessThanTen(this.month + 1)}-01`).getTime()
      );
    },

    daysCount: function() {
      return utils.daysInMonth(this.year, this.month);
    }
  },
  data() {
    const { month, startDate, endDate, singleDate } = this;
    return {
      selectedDay: null,
      isSelectingStartDay: true, // either startDay or endDay
      weekdays: utils.weekDayShortConfig,
      innerStartDate: startDate,
      innerEndDate: singleDate ? startDate : endDate
    };
  }
};
</script>

<style lang="scss" scoped>
@import "../../style/main";

ul.tvh-calendar {
  width: 364px;
  display: flex;
  flex-wrap: wrap;
  justify-content: flex-start;
  background: #fff;

  li {
    display: inline-block;
    width: 52px;
  }
  li.tvh-weekday {
    font-size: 14px;
    color: $silver;
    font-weight: 600;
    margin-bottom: 8px;
    text-align: center;
  }
  li.tvh-day {
    span {
      width: 100%;
      height: 40px;
      display: inline-block;
      text-align: center;
      line-height: 40px;
      font-size: 15px;
      font-weight: 600;
      color: $slate-grey;
      background: #fff;

      &:hover {
        cursor: pointer;
        color: #fff;
        background: $secondary-01;
        transition-duration: 0.3s;
      }
      &.tvh-today {
        box-shadow: inset 0 0 0 2px $secondary-01;
      }
      &.tvh-innerStartDate {
        background: $secondary-01;
        color: #fff;
      }
      &.tvh-innerEndDate {
        background: $secondary-01;
        color: #fff;
      }
      &.tvh-between {
        background: #eaf0fd;
      }

      &.tvh-disabled {
        pointer-events: none;
        background: #f9f9f9;
        color: #ccc;
      }
    }
  }
}
@media only screen and (max-width: 700px) {
  ul.tvh-calendar {
    width: 100%;
    li {
      width: calc(100% / 7);
    }
  }
}
</style>
