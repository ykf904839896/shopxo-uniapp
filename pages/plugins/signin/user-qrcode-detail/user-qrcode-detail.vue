<template>
    <view>
        <view v-if="detail != null">
            <view class="padding-horizontal-main padding-top-main">
                <view v-if="detail_list.length > 0" class="panel-item padding-main border-radius-main bg-white spacing-mb">
                    <view class="br-b padding-bottom-main fw-b text-size">基础信息</view>
                    <view class="panel-content oh">
                        <view v-for="(item, index) in detail_list" :key="index" class="item br-b oh padding-vertical-main">
                            <view class="title fl padding-right-main cr-gray">{{item.name}}</view>
                            <view class="content fl br-l padding-left-main">{{item.value}}</view>
                        </view>
                    </view>
                </view>
            
                <!-- 连续签到翻倍奖励 -->
                <view v-if="(detail.continuous_rules || null) != null && detail.continuous_rules.length > 0" class="panel-item padding-main border-radius-main bg-white spacing-mb">
                    <view class="br-b padding-bottom-main fw-b text-size">连续签到翻倍奖励</view>
                    <view class="panel-content oh">
                        <view v-for="(item, index) in detail.continuous_rules" :key="index" class="item br-b oh padding-vertical-main">
                            <view class="content fl">连续{{item.number}}天，翻{{item.value}}倍</view>
                        </view>
                    </view>
                </view>
                
                <!-- 指定时段额外奖励 -->
                <view v-if="(detail.specified_time_reward || null) != null && (detail.specified_time_reward.time_start || null) != null && (detail.specified_time_reward.time_end || null) != null && (detail.specified_time_reward.value || null) != null" class="panel-item padding-main border-radius-main bg-white spacing-mb">
                    <view class="br-b padding-bottom-main fw-b text-size">指定时段额外奖励</view>
                    <view class="panel-content oh">
                        <view v-for="(item, index) in detail.specified_time_reward" :key="index" class="item br-b oh padding-vertical-main">
                            <view class="content fl">时段 {{detail.specified_time_reward.time_start}} - {{detail.specified_time_reward.time_end}}，额外奖励 {{detail.specified_time_reward.value}}</view>
                        </view>
                    </view>
                </view>
            </view>
        
            <!-- 结尾 -->
            <component-bottom-line :propStatus="data_bottom_line_status"></component-bottom-line>
        </view>
        <view v-else>
            <!-- 提示信息 -->
            <component-no-data :propStatus="data_list_loding_status" :propMsg="data_list_loding_msg"></component-no-data>
        </view>
    </view>
</template>
<script>
    const app = getApp();
    import componentNoData from "../../../../components/no-data/no-data";
    import componentBottomLine from "../../../../components/bottom-line/bottom-line";

    export default {
        data() {
            return {
                params: null,
                data_list_loding_status: 1,
                data_list_loding_msg: '',
                data_bottom_line_status: false,
                detail: null,
                detail_list: []
            };
        },

        components: {
            componentNoData,
            componentBottomLine
        },
        props: {},

        onLoad(params) {
            //params['id'] = 1;
            this.setData({
                params: params
            });
            this.init();
        },

        onShow() {
            // 分享菜单处理
            app.globalData.page_share_handle();
        },

        // 下拉刷新
        onPullDownRefresh() {
            this.init();
        },

        methods: {
            init() {
                uni.showLoading({
                    title: "加载中..."
                });
                this.setData({
                    data_list_loding_status: 1
                });
                uni.request({
                    url: app.globalData.get_request_url("detail", "userqrcode", "signin"),
                    method: "POST",
                    data: {
                        id: this.params.id
                    },
                    dataType: "json",
                    success: res => {
                        uni.hideLoading();
                        uni.stopPullDownRefresh();
                        if (res.data.code == 0) {
                            var data = res.data.data;
                            this.setData({
                                detail: data.data,
                                detail_list: [
                                    { name: "是否启用", value: data.data.is_enable_name || '' },
                                    { name: "邀请人奖励", value: (data.data.reward_master || 0)+' 积分' },
                                    { name: "受邀人奖励", value: (data.data.reward_invitee || 0)+' 积分' },
                                    { name: "联系人姓名", value: data.data.name || '' },
                                    { name: "联系人电话", value: data.data.tel || '' },
                                    { name: "联系人地址", value: data.data.address || '' },
                                    { name: "创建时间", value: data.data.add_time || '' },
                                    { name: "更新时间", value: data.data.upd_time || '' },
                                ],
                                data_list_loding_status: 3,
                                data_bottom_line_status: true,
                                data_list_loding_msg: ''
                            });
                        } else {
                            this.setData({
                                data_list_loding_status: 2,
                                data_bottom_line_status: false,
                                data_list_loding_msg: res.data.msg
                            });
                            if (app.globalData.is_login_check(res.data, this, 'init')) {
                                app.globalData.showToast(res.data.msg);
                            }
                        }
                    },
                    fail: () => {
                        uni.hideLoading();
                        uni.stopPullDownRefresh();
                        this.setData({
                            data_list_loding_status: 2,
                            data_bottom_line_status: false,
                            data_list_loding_msg: '服务器请求出错'
                        });
                        app.globalData.showToast("服务器请求出错");
                    }
                });
            }
        }
    };
</script>
<style>
</style>