<template>
    <view>
        <scroll-view :scroll-y="true" class="scroll-box" @scrolltolower="scroll_lower" lower-threshold="30">
            <view class="data-list">
                <view v-if="data_list.length > 0" class="data-list padding-horizontal-main padding-top-main">
                    <view v-for="(item, index) in data_list" :key="index" class="item padding-main border-radius-main oh bg-white spacing-mb">
                        <view class="base oh br-b padding-bottom-main">
                            <image class="select-icon va-m margin-right-sm" data-type="node" :data-value="item.id" @tap="selected_event" :src="common_static_url+'select' + (select_ids.indexOf(item.id) != -1 ? '-active' : '') + '-icon.png'" mode="widthFix"></image>
                            <text class="cr-base va-m">{{item.add_time}}</text>
                        </view>
                        <view class="content margin-top">
                            <navigator :url="'/pages/user-order-detail/user-order-detail?id=' + item.id" hover-class="none">
                                <block v-for="(fv,fi) in content_list">
                                    <view class="single-text margin-top-xs">
                                        <text class="cr-gray margin-right-xl">{{fv.name}}</text>
                                        <text class="cr-base">{{item[fv.field]}}</text>
                                        <text v-if="(fv.unit || null) != null" class="cr-gray">{{fv.unit}}</text>
                                    </view>
                                </block>
                            </navigator>
                        </view>
                        <view class="item-operation tr br-t padding-top-main margin-top-main">
                            <button class="round bg-white br cr-base br" type="default" size="mini" hover-class="none" :data-ids="item.id" data-type="item" @tap="invoice_event">开票</button>
                        </view>
                    </view>

                    <!-- 合并开票 -->
                    <button v-if="select_ids.length > 0" class="bottom-fixed invoice-merge-submit bg-green cr-white round" type="default" size="mini" hover-class="none" data-type="all" @tap="invoice_event">合并开票</button>
                </view>
                <view v-else>
                    <!-- 提示信息 -->
                    <component-no-data :propStatus="data_list_loding_status"></component-no-data>
                </view>

                <!-- 结尾 -->
                <component-bottom-line :propStatus="data_bottom_line_status"></component-bottom-line>
            </view>
        </scroll-view>
    </view>
</template>
<script>
    const app = getApp();
    import componentNoData from "../../../../components/no-data/no-data";
    import componentBottomLine from "../../../../components/bottom-line/bottom-line";

    var common_static_url = app.globalData.get_static_url('common');
    export default {
        data() {
            return {
                common_static_url: common_static_url,
                data_base: null,
                data_list: [],
                data_total: 0,
                data_page_total: 0,
                data_page: 1,
                data_list_loding_status: 1,
                data_bottom_line_status: false,
                params: null,
                select_ids: [],
                content_list: [
                    {name: "订单编号", field: "order_no"},
                    {name: "订单总额", field: "total_price", unit: "元"},
                    {name: "支付金额", field: "pay_price", unit: "元"},
                    {name: "订单单价", field: "price", unit: "元"}
                ]
            };
        },

        components: {
            componentNoData,
            componentBottomLine
        },
        props: {},

        onLoad(params) {
            this.setData({
                params: params
            });
        },

        onShow() {
            this.init();

            // 分享菜单处理
            app.globalData.page_share_handle();
        },

        // 下拉刷新
        onPullDownRefresh() {
            this.setData({
                data_page: 1
            });
            this.get_data_list(1);
        },

        methods: {
            init() {
                var user = app.globalData.get_user_info(this, 'init');
                if (user != false) {
                    // 用户未绑定用户则转到登录页面
                    if (app.globalData.user_is_need_login(user)) {
                        uni.redirectTo({
                            url: "/pages/login/login?event_callback=init"
                        });
                        return false;
                    } else {
                        // 获取数据
                        this.get_data_list();
                    }
                } else {
                    this.setData({
                        data_list_loding_status: 0,
                        data_bottom_line_status: false
                    });
                }
            },

            // 获取数据
            get_data_list(is_mandatory) {
                // 分页是否还有数据
                if ((is_mandatory || 0) == 0) {
                    if (this.data_bottom_line_status == true) {
                        uni.stopPullDownRefresh();
                        return false;
                    }
                }
                
                // 加载loding
                uni.showLoading({
                    title: "加载中..."
                });
                this.setData({
                    data_list_loding_status: 1
                });

                // 获取数据
                uni.request({
                    url: app.globalData.get_request_url("index", "order", "invoice"),
                    method: "POST",
                    data: {
                        page: this.data_page
                    },
                    dataType: "json",
                    success: res => {
                        uni.hideLoading();
                        uni.stopPullDownRefresh();
                        if (res.data.code == 0) {
                            if (res.data.data.data.length > 0) {
                                if (this.data_page <= 1) {
                                    var temp_data_list = res.data.data.data;
                                } else {
                                    var temp_data_list = this.data_list || [];
                                    var temp_data = res.data.data.data;
                                    for (var i in temp_data) {
                                        temp_data_list.push(temp_data[i]);
                                    }
                                }
                                this.setData({
                                    data_base: res.data.data.base || null,
                                    data_list: temp_data_list,
                                    data_total: res.data.data.total,
                                    data_page_total: res.data.data.page_total,
                                    data_list_loding_status: 3,
                                    data_page: this.data_page + 1
                                });
                                
                                // 是否还有数据
                                this.setData({
                                    data_bottom_line_status: (this.data_page > 1 && this.data_page > this.data_page_total)
                                });
                            } else {
                                this.setData({
                                    data_list_loding_status: 0,
                                    data_list: [],
                                    data_bottom_line_status: false
                                });
                            }
                        } else {
                            this.setData({
                                data_list_loding_status: 0
                            });
                            if (app.globalData.is_login_check(res.data, this, 'get_data_list')) {
                                app.globalData.showToast(res.data.msg);
                            }
                        }
                    },
                    fail: () => {
                        uni.hideLoading();
                        uni.stopPullDownRefresh();
                        this.setData({
                            data_list_loding_status: 2
                        });
                        app.globalData.showToast("服务器请求出错");
                    }
                });
            },

            // 滚动加载
            scroll_lower(e) {
                this.get_data_list();
            },

            // 选择
            selected_event(e) {
                var value = e.currentTarget.dataset.value;
                var temp_select_ids = this.select_ids;
                var index = temp_select_ids.indexOf(value);
                if (index == -1) {
                    temp_select_ids.push(value);
                } else {
                    temp_select_ids.splice(index, 1);
                }
                this.setData({
                    select_ids: temp_select_ids
                });
            },

            // 合并开票
            invoice_event(e) {
                var type = e.currentTarget.dataset.type || 'all';
                var ids = e.currentTarget.dataset.ids || null;
                if(type == 'all') {
                    if (this.select_ids.length <= 0) {
                        app.globalData.showToast('请先选择数据');
                        return false;
                    } else {
                        ids = this.select_ids.join(',');
                    }
                } else {
                    if(ids === null) {
                        app.globalData.showToast('元素参数id有误');
                        return false;
                    }
                }
                uni.navigateTo({
                    url: '/pages/plugins/invoice/invoice-saveinfo/invoice-saveinfo?ids=' + ids + '&type=order&is_redirect=1'
                });
            }
        }
    };
</script>
<style>
    @import './order.css';
</style>