<template>
    <view>
        <view class="time-select-content" @click="_dataOpen">
            <slot></slot>
        </view>
        <view :class="'time-select-popup-mask '+(propIsShow ? 'time-select-popup-show' : '')" @click="_maskClose">
            <view class="time-select-popup-content" @click.stop="_stopFunc">
                <view class="time-select-close-btn" v-if="propCloseBtn" @click="_closeBtnClose">×</view>
                <view class="time-select-title">
                    <view v-if="(propTitle || null) != null">{{ propTitle }}</view>
                    <view v-if="(propSubhead || null) != null">{{ propSubhead }}</view>
                </view>
                <view class="time-select-time-box">
                    <view class="left_box">
                        <view v-if="item.timeArr.length > 0" @click="_changeDay(index)" :class="{ active: item.checked }" v-for="(item, index) in timeList" :key="item.dateStr">
                            {{ item.name }}
                        </view>
                    </view>
                    <view class="right_box">
                        <view @click="_changeTime(index)" :class="{ active: item.checked }" v-for="(item, index) in activeTimeArr" :key="item.time">
                            {{ item.time }}{{ propRangeType ? '-' + item.endtime : '' }}
                        </view>
                    </view>
                </view>
            </view>
        </view>
    </view>
</template>

<script>
export default {
    props: {
        propTitle: {
            type: String,
            default: '请选择时间'
        },
        propSubhead: {
            type: String,
            default: ''
        },
        propRangeType: {
            type: Boolean,
            default: false
        },
        propIsShow: {
            type: Boolean,
            default: false
        },
        propMaskHide: {
            type: Boolean,
            default: true
        },
        propCloseBtn: {
            type: Boolean,
            default: true
        },
        propRangeDay: {
            type: [Number, String],
            default: 2
        },
        propRangeStartTime: {
            type: String,
            default: '00:00:00'
        },
        propRangeEndTime: {
            type: String,
            default: '00:00:00'
        },
        propDefaultTime: {
            type: String,
            default: ''
        },
        propIsRoundingTime: {
            type: Boolean,
            default: true
        },
        propIntervalTime: {
            //间隔时间
            type: [Number, String],
            default: 30
        },
        propIsNow:{
            type: Boolean,
            default: false
        },
        propDayStartIntTime: {
            //每天开始间隔时间
            type: [Number, String],
            default: 0
        },
        propDisabled: {
            type: [String, Array],
            default: () => []
        }
    },
    data() {
        return {
            timeList: [],
            selectDateStr: '',
            select_dateStr: '',
            selectTime: '',
            selectEndime: '',
            activeTimeArr: []
        };
    },
    beforeMount() {
        this._initDay();
    },
    watch: {
        propIsShow: function(val, oldVal) {
            val && this.timeList.length <= 0 && this._initDay();
        }
    },
    methods: {
        _stopFunc() {},

        _dataOpen() {
            this._selectEvent();
        },
        _closeBtnClose() {
            if(this.propCloseBtn) {
                this._selectEvent();
            }
        },
        _maskClose() {
            if(this.propMaskHide) {
                this._selectEvent();
            }
        },
        _selectEvent(data = '') {
            this.$emit('selectEvent', data);
        },
        _changeDay(e) {
            let _ind = e - 0;
            let { timeList } = this;
            timeList.forEach(ele => {
                ele.checked = false;
            });
            timeList[_ind].checked = true;
            this.timeList = timeList;
            this.selectDateStr = timeList[_ind].dateStr;
            this.select_dateStr = timeList[_ind]._dateStr;
            this.activeTimeArr = timeList[_ind].timeArr;
        },
        _changeTime(e) {
            let _ind = e - 0;
            let { activeTimeArr } = this;
            let timeArr = JSON.parse(JSON.stringify(activeTimeArr));
            timeArr.forEach(ele => {
                ele.checked = false;
            });
            timeArr[_ind].checked = true;
            this.selectTime = timeArr[_ind].time;
            this.selectEndime = timeArr[_ind].endtime;
            this.activeTimeArr = timeArr;
            let _data = this._handleData();
            this._selectEvent(_data);
        },
        _handleData() {
            let _data = {};
            let { selectDateStr, select_dateStr, selectTime, selectEndime } = this;
            _data.date = selectDateStr + ' ' + selectTime;
            _data._date = select_dateStr + ' ' + selectTime;
            _data.dateRange = selectDateStr + ' ' + selectTime + '-' + selectEndime;
            _data._dateRange = select_dateStr + ' ' + selectTime + '-' + selectEndime;
            _data.timeStamp = new Date(selectDateStr + ' ' + selectTime).getTime();
            return _data;
        },
        _initDay() {
            let _timeList = [];
            for (let index = 0; index < this.propRangeDay; index++) {
                let _item = {
                    ...this._getDate(index)
                };
                _item.timeArr = this._initTime(index);
                _timeList.push(_item);
            }

            if (this.propDefaultTime) {
                //存在默认时间
                let _day = this.propDefaultTime.split(' ')[0].replace(/-/g, '/');
                let _time = this.propDefaultTime.split(' ')[1];
                let _flag = true;
                for (let index = 0; index < _timeList.length; index++) {
                    const element = _timeList[index];
                    element.checked = false;
                    if (element.timeArr.length > 0 && element.dateStr === _day) {
                        element.checked = true;
                        _flag = false;
                        element.timeArr.forEach(item => {
                            if (this._timeRange(item.time + ':00', item.endtime + ':00', _time)) {
                                item.checked = true;
                                this.time = item.time;
                                this.endtime = item.endtime;
                            }
                        });

                        this.selectDateStr = element.dateStr;
                        this.select_dateStr = element._dateStr;
                        this.activeTimeArr = element.timeArr;
                    }
                }
                if (_flag) {
                    this._setDefaultTime(_timeList);
                }
            } else {
                this._setDefaultTime(_timeList);
            }

            this.timeList = _timeList;
        },
        _setDefaultTime(list) {
            for (let index = 0; index < list.length; index++) {
                const element = list[index];
                if (element.timeArr.length > 0) {
                    element.checked = true;
                    //是否默认选择时间
                    // element.timeArr[0].checked = true;
                    this.selectDateStr = element.dateStr;
                    this.select_dateStr = element._dateStr;
                    this.activeTimeArr = element.timeArr;
                    break;
                }
            }
        },
        _initTime(num) {
            let _item = [];
            let TempDisabled = this.propDisabled;
            let tempIntervalTime = this.propIntervalTime;
            if(tempIntervalTime<=0){
                tempIntervalTime = 1
            }
            let TempRangeStartTime = this._timeHandle(this.propRangeStartTime);
            let TempRangeEndTime = this._timeHandle(this.propRangeEndTime);
            if(TempRangeEndTime == '00:00:00') {
                TempRangeEndTime = '23:59:59';
            }
            if (typeof TempDisabled === 'string') {
                TempDisabled = TempDisabled ? TempDisabled.split(',') : [];
            } else if (Array.isArray(TempDisabled)) {
                TempDisabled = TempDisabled.map(ele => {
                    return ele + '';
                });
            }
            if (num === 0 && !TempDisabled.includes('0')) {
                //今天
                let _nowTime = this._roundingTime();
                if (this._timeRange(TempRangeStartTime, TempRangeEndTime, _nowTime)) {
                    //当前时间在营业时间内
                    return this._forTime(_nowTime, TempRangeEndTime, tempIntervalTime, _nowTime);
                } else if (this._toTimeStr(_nowTime) < this._toTimeStr(TempRangeStartTime)) {
                    //早于今天开始时间
                    return this._forTime(TempRangeStartTime, TempRangeEndTime, tempIntervalTime, _nowTime);
                } else {
                    return [];
                }
            } else {
                //其他日期
                if (TempDisabled.includes(num + '')) {
                    //禁止当前日期了
                    return [];
                }

                return this._forTime(TempRangeStartTime, TempRangeEndTime, tempIntervalTime);
            }
        },
        _getDate(num) {
            let date = new Date();
            let date1 = new Date(date);
            let weekday = ['周日', '周一', '周二', '周三', '周四', '周五', '周六'];
            date1.setDate(date.getDate() + num);
            let name = '',
                dateStr = '';
            name = date1.getMonth() - 0 + 1 + '月' + date1.getDate() + '日(' + weekday[date1.getDay()] + ')';
            dateStr = date1.getFullYear() + '/' + (date1.getMonth() - 0 + 1) + '/' + date1.getDate();
            if (num == 0) {
                name = '今天(' + weekday[date1.getDay()] + ')';
            } else if (num == 1) {
                name = '明天(' + weekday[date1.getDay()] + ')';
            }
            return {
                name: name,
                dateStr: dateStr,
                _dateStr: dateStr.replace(/\//g, '-')
            };
        },
        _addZero(num) {
            num = num + '';
            if (num.length === 1) {
                return '0' + num;
            }
            return num;
        },
        _roundingTime() {
            let _now = new Date();
            let _h = _now.getHours() - 0;
            let _m = _now.getMinutes() - 0;
            if (_m > 30) {
                _m = '00';
                _h += 1;
            }else if (_m - 0 > 0&&_m - 0<30) {
                _m = 30;
            }
            if (this.propIsRoundingTime&&!this.propIsNow) {
                return _h + ':' + _m + ':00';
            }
            return new Date().getHours() + ':' + new Date().getMinutes() + ':00';
        },
        _forTime(st, et, it, dt = 0) {
            let TempDayStartIntTime = this.propDayStartIntTime * 1000 * 60;
            let _st = this._toTimeStr(st);
            let _et = this._toTimeStr(et);
            let _it = it * 1000 * 60;
            let _list = [];
            if (_st < _et) {
                for (let i = _st + TempDayStartIntTime; i < _et; i += _it) {
                    _list.push({
                        time: this._toLocalTime(i),
                        endtime: this._toLocalTime(i + _it > _et ? _et : i + _it),
                        checked: false
                    });
                }
            } else {
                //跨天了
                for (let i = dt; i < _et; i += _it) {
                    _list.push({
                        time: this._toLocalTime(i),
                        endtime: this._toLocalTime(i + _it > _et ? _et : i + _it),
                        checked: false
                    });
                }
                for (let i = _st  + TempDayStartIntTime; i < this._toTimeStr('23:59:59'); i += _it) {
                    _list.push({
                        time: this._toLocalTime(i),
                        endtime: this._toLocalTime(i + _it),
                        checked: false
                    });
                }
            }
            return _list;
        },
        _toTimeStr(time = '') {
            let newTime = time;
            let itemStr = 0;
            let timeArr = newTime.split(':');
            let h = timeArr[0] * 1000 * 60 * 60;
            let m = timeArr[1] * 1000 * 60;
            let s = timeArr[2] * 1000;
            itemStr = h + m + s;
            return itemStr;
        },
        _toLocalTime(v, type = 0) {
            let h = parseInt((v / 1000 / 60 / 60) % 24);
            let m = parseInt((v / 1000 / 60) % 60);
            let s = parseInt((v / 1000) % 60);
            v = type === 0 ? `${this._addZero(h)}:${this._addZero(m)}` : `${this._addZero(h)}:${this._addZero(m)}:${this._addZero(s)}`;
            return v;
        },
        _timeRange(beginTime, endTime, nowTime) {
            let strb = beginTime.split(':');
            let stre = endTime.split(':');
            let strn = nowTime.split(':');

            let b = new Date();
            let e = new Date();
            let n = new Date();

            b.setHours(strb[0]);
            b.setMinutes(strb[1]);
            e.setHours(stre[0]);
            e.setMinutes(stre[1]);
            n.setHours(strn[0]);
            n.setMinutes(strn[1]);

            let _b = b.getTime();
            let _e = e.getTime();
            let _n = n.getTime();

            if (_b > _e) {
                if (_n - _e >= 0 && _n - _b < 0) {
                    return false;
                } else {
                    return true;
                }
            } else {
                if (_n - _b >= 0 && _n - _e < 0) {
                    return true;
                } else {
                    return false;
                }
            }
        },
        _timeHandle(value) {
            let arr = ((value || null) == null) ? [] : value.split(':');
            let len = arr.length;
            if(len < 3) {
                while(len<3) {
                    arr.push('00');
                    len++;
                }
            }
            return arr.join(':');
        }
    }
};
</script>
<style>
.time-select-popup-mask {
    position: fixed;
    left: 0;
    right: 0;
    bottom: 0;
    top: 0;
    background-color: rgba(0, 0, 0, 0.5);
    opacity: 0;
    z-index: 100;
    pointer-events: none;
}
.time-select-popup-mask view {
    box-sizing: border-box;
}
.time-select-popup-content {
    height: 0rpx;
    background-color: #fff;
    position: absolute;
    bottom: 0;
    left: 0;
    width: 100%;
    padding: 40rpx 10rpx 0 10rpx;
    transition: height 0.5s;
    border-top-right-radius: 20rpx;
    border-top-left-radius: 20rpx;
    height: auto;
    z-index: 101;
    transform: translateY(100%);
}
.time-select-popup-show {
    opacity: 1;
    pointer-events: auto;
}
.time-select-popup-show .time-select-popup-content {
    transform: none;
}
.time-select-popup-mask,
.time-select-popup-content {
    transition: all 0.25s linear;
}
.time-select-title {
    text-align: center;
    font-size: 32rpx;
    color: #222222;
    font-weight: 600;
    position: relative;
}
.time-select-close-btn {
    position: absolute;
    width: 80rpx;
    line-height: 80rpx;
    text-align: center;
    right: 0rpx;
    padding-right: 15rpx;
    top: 0;
    color: #999999;
    z-index: 99;
    font-size: 44rpx;
}
.time-select-title > view:nth-child(2) {
    font-size: 22rpx;
    color: #919191;
    margin-top: 10rpx;
}
.time-select-time-box {
    border-top: 1px solid #fbf8fb;
    background-color: #fbf8fb;
    height: 660rpx;
    display: flex;
    justify-content: space-between;
    align-items: center;
}
.time-select-time-box > view {
    height: 100%;
    overflow-y: auto;
    -webkit-overflow-scrolling: touch;
    font-size: 28rpx;
}
.time-select-time-box .left_box {
    color: #919191;
    width: 45%;
}
.time-select-time-box .left_box view,
.time-select-time-box .right_box view {
    text-align: center;
    line-height: 90rpx;
    position: relative;
    color: #666;
}
.time-select-time-box .right_box view::before {
    content: ' ';
    width: 80%;
    position: absolute;
    left: 10%;
    bottom: 0;
    border: 1px dashed #f4f4f4;
}
.time-select-time-box .right_box view:last-child::before {
    border: none;
}
.time-select-time-box .right_box {
    width: 55%;
    flex: 1;
    background-color: #fff;
    text-align: center;
}
.time-select-time-box .right_box .active {
    position: relative;
    color: #000;
    font-weight: bold;
}
.time-select-time-box .left_box .active {
    background-color: #fff;
    color: #000;
}
.time-select-time-box .right_box .active::after {
    content: ' ';
    position: absolute;
    top: 50%;
    margin-top: -16rpx;
    right: 50rpx;
    width: 10rpx;
    height: 20rpx;
    border-color: #000;
    border-style: solid;
    border-width: 0 4rpx 4rpx 0;
    transform: rotate(45deg);
}
</style>