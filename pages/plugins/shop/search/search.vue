<template>
    <view>
        <!-- 排序 -->
        <view class="nav-sort bg-white oh pr">
            <view class="nav-sort-content">
                <block v-for="(item, index) in search_nav_sort_list" :key="index">
                    <view class="item tc fl" :data-index="index" @tap="nav_sort_event">
                        <text class="cr-base va-m">{{item.name}}</text>
                        <image v-if="(item.icon || null) != null" class="icon va-m" :src="common_static_url + 'sort-' + item.icon + '-icon.png'" mode="aspectFill"></image>
                    </view>
                </block>
            </view>
            <image class="screening-submit pa" :src="common_static_url+'search-submit-icon.png'" mode="aspectFill" @tap="popup_form_event_show"></image>
        </view>

        <!-- 列表 -->
        <scroll-view :scroll-y="true" class="scroll-box scroll-box-ece-nav" @scrolltolower="scroll_lower" lower-threshold="30">
            <view v-if="data_list.length > 0" class="data-list padding-horizontal-main padding-top-main oh">
                <view v-for="(item, index) in data_list" :key="index" class="item padding-bottom-sm border-radius-main bg-white margin-bottom-main oh">
                    <navigator :url="'/pages/goods-detail/goods-detail?id=' + item.id" hover-class="none">
                        <image class="goods-img dis-block" :src="item.images" mode="aspectFit"></image>
                        <view class="base padding-horizontal-main margin-top">
                            <view class="multi-text">{{item.title}}</view>
                            <view class="price margin-top">
                                <text class="sales-price">{{currency_symbol}}{{item.min_price}}</text>
                            </view>
                        </view>
                    </navigator>
                </view>
            </view>
            <view v-else>
                <!-- 提示信息 -->
                <component-no-data :propStatus="data_list_loding_status"></component-no-data>
            </view>

            <!-- 结尾 -->
            <component-bottom-line :propStatus="data_bottom_line_status"></component-bottom-line>
        </scroll-view>

        <!-- 筛选条件 popup -->
        <component-popup :propShow="is_show_popup_form" propPosition="left" @onclose="popup_form_event_close">
            <form @submit="form_submit_event" class="popup-form oh">
                <view class="search-map padding-main bg-base">
                    <view class="padding-main border-radius-main bg-white">
                        <view class="map-item map-base br-b">
                            <text>筛选出</text>
                            <text class="cr-main"> {{data_total}} </text>
                            <text>条数据</text>
                            <text class="fr cr-red" @tap="map_remove_event">清除</text>
                        </view>
                        <!-- 搜索关键字 -->
                        <input type="text" confirm-type="done" placeholder="其实搜索很简单^_^ !" name="wd" :value="(post_data.wd || '')" class="map-keywords wh-auto round bg-base margin-top-lg" placeholder-class="cr-grey">
                    </view>

                    <!-- 分类 -->
                    <view v-if="(search_map_list.category_list || null) != null && search_map_list.category_list.length > 0" class="map-item padding-horizontal-main padding-top-main border-radius-main bg-white spacing-mt">
                        <view class="map-nav pr br-b">
                            <text>分类</text>
                            <text class="arrow-bottom pa cr-grey" v-if="search_map_list.category_list.length > 3" @tap="more_event" data-value="category_list">更多</text>
                        </view>
                        <view class="map-content map-text-item map-category-container oh margin-top-lg" :style="'height:' + map_fields_list.category_list.height + ';'">
                            <block v-for="(item, index) in search_map_list.category_list" :key="index">
                                <view :class="'item fl cr-base radius ' + (item.active == 1 ? 'cr-main br-main' : '')" @tap="map_item_event" :data-index="index" data-field="category_list">{{item.name}}</view>
                            </block>
                        </view>
                    </view>

                    <view class="search-submit padding-main pa">
                        <button form-type="submit" class="bg-main cr-white text-size wh-auto round" :disabled="popup_form_loading_status" hover-class="none">确认</button>
                    </view>
                </view>
            </form>
        </component-popup>
    </view>
</template>
<script>
    const app = getApp();
    import componentPopup from "../../../../components/popup/popup";
    import componentNoData from "../../../../components/no-data/no-data";
    import componentBottomLine from "../../../../components/bottom-line/bottom-line";

    var common_static_url = app.globalData.get_static_url('common');
    export default {
        data() {
            return {
                common_static_url: common_static_url,
                data_list_loding_status: 1,
                data_bottom_line_status: false,
                currency_symbol: app.globalData.data.currency_symbol,
                data_list: [],
                data_total: 0,
                data_page_total: 0,
                data_page: 1,
                params: null,
                post_data: {},
                shop: null,
                is_show_popup_form: false,
                popup_form_loading_status: false,
                search_nav_sort_list: [
                    { name: "综合", field: "default", sort: "asc", "icon": null },
                    { name: "销量", field: "sales_count", sort: "asc", "icon": "default" },
                    { name: "热度", field: "access_count", sort: "asc", "icon": "default" },
                    { name: "价格", field: "min_price", sort: "asc", "icon": "default" },
                    { name: "最新", field: "id", sort: "asc", "icon": "default" }
                ],
                // 搜素条件
                search_map_list: {
                    category_list: []
                },
                map_fields_list: {
                    category_list: {height: "82rpx", default: "82rpx", form_key: "category_ids"}
                },
                // 自定义分享信息
                share_info: {}
            };
        },

        components: {
            componentPopup,
            componentNoData,
            componentBottomLine
        },
        props: {},

        onLoad(params) {
            this.setData({
                params: params,
                post_data: {
                    wd: params.keywords || '',
                    shop_id: params.shop_id || 0,
                    category_ids: ((params.category_id || 0) == 0) ? '' : JSON.stringify({"0":params.category_id})
                }
            });
        },

        onShow() {
            // 初始化配置
            this.init_config();

            // 数据加载
            this.get_data();
        },

        // 下拉刷新
        onPullDownRefresh() {
            this.setData({
                data_page: 1
            });
            this.get_data_list(1);
        },

        methods: {
            // 初始化配置
            init_config(status) {
                if ((status || false) == true) {
                    this.setData({
                        currency_symbol: app.globalData.get_config('currency_symbol')
                    });
                } else {
                    app.globalData.is_config(this, 'init_config');
                }
            },

            // 搜索
            search_event() {
                this.setData({
                    data_list: [],
                    data_page: 1
                });
                this.get_data_list(1);
            },
            
            // 初始化
            get_data() {
                uni.showLoading({
                    title: "加载中...",
                    mask: true
                });
                var post_data = this.request_map_handle();
                uni.request({
                    url: app.globalData.get_request_url("index", "search", "shop"),
                    method: "POST",
                    data: post_data,
                    dataType: "json",
                    success: res => {
                        uni.hideLoading();
                        uni.stopPullDownRefresh();
                        if (res.data.code == 0) {
                            var data = res.data.data;
                            // 分类选中处理
                            var category = data.shop_goods_category || [];
                            if((this.params.category_id || 0) != 0 && category.length > 0) {
                                for(var i in category) {
                                    category[i]['active'] = (category[i]['id'] == this.params.category_id) ? 1 : 0;
                                }
                            }
                            this.setData({
                                shop: data.shop || null,
                                search_map_info: data.search_map_info || [],
                                search_map_list: {
                                    category_list: category
                                }
                            });

                            if((this.shop || null) != null) {
                                // 基础自定义分享
                                var shop_id = this.shop.id;
                                var category_id = this.params.category_id || 0;
                                var keywords = this.params.keywords || '';
                                this.setData({
                                    share_info: {
                                        title: this.shop.seo_title || this.shop.name,
                                        desc: this.shop.seo_desc || this.shop.describe,
                                        path: '/pages/plugins/shop/search/search',
                                        query: 'shop_id='+shop_id+'&category_id='+category_id+'&keywords='+keywords,
                                        img: this.shop.logo
                                    }
                                });

                                // 获取列表数据
                                this.get_data_list(1);
                            } else {
                                this.setData({
                                    data_list_loding_status: 0,
                                    data_bottom_line_status: false
                                });
                            }
                        } else {
                            this.setData({
                                data_list_loding_status: 0
                            });
                            app.globalData.showToast(res.data.msg);
                        }

                        // 分享菜单处理
                        app.globalData.page_share_handle(this.share_info);
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

            // 获取数据列表
            get_data_list(is_mandatory) {
                // 分页是否还有数据
                if ((is_mandatory || 0) == 0) {
                    if (this.data_bottom_line_status == true) {
                        uni.stopPullDownRefresh();
                        return false;
                    }
                }
                
                // 获取数据
                uni.showLoading({
                    title: "加载中...",
                    mask: true
                });
                var post_data = this.request_map_handle();
                uni.request({
                    url: app.globalData.get_request_url("datalist", "search", "shop"),
                    method: "POST",
                    data: post_data,
                    dataType: "json",
                    success: res => {
                        uni.hideLoading();
                        uni.stopPullDownRefresh();
                        if (res.data.code == 0) {
                            var data = res.data.data;
                            if (data.data.length > 0) {
                                if (this.data_page <= 1) {
                                    var temp_data_list = data.data;
                                } else {
                                    var temp_data_list = this.data_list || [];
                                    var temp_data = data.data;
                                    for (var i in temp_data) {
                                        temp_data_list.push(temp_data[i]);
                                    }
                                }
                                this.setData({
                                    data_list: temp_data_list,
                                    data_total: data.total,
                                    data_page_total: data.page_total,
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
                                    data_total: 0
                                });
                                if (this.data_page <= 1) {
                                    this.setData({
                                        data_list: [],
                                        data_bottom_line_status: false
                                    });
                                }
                            }
                        } else {
                            this.setData({
                                data_list_loding_status: 0
                            });
                            app.globalData.showToast(res.data.msg);
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

            // 搜索条件处理
            request_map_handle() {
                var params = this.params;
                var post_data = this.post_data;
                post_data['page'] = this.data_page;
                
                // 店铺id
                post_data['shop_id'] = params['shop_id'] || 0;

                // 搜索条件
                var temp_field = this.map_fields_list;
                var temp_list = this.search_map_list;
                for (var i in temp_field) {
                    if (temp_list[i] != null != null && temp_list[i].length > 0) {
                        var temp = {};
                        var index = 0;
                        for (var k in temp_list[i]) {
                            if ((temp_list[i][k]['active'] || 0) == 1) {
                                temp[index] = temp_list[i][k]['id'];
                                index++;
                            }
                        }
                        post_data[temp_field[i]['form_key']] = app.globalData.get_length(temp) > 0 ? JSON.stringify(temp) : '';
                    }
                }
                return post_data;
            },

            // 滚动加载
            scroll_lower(e) {
                this.get_data_list();
            },

            // 搜索条件
            form_submit_event(e) {
                this.setData({
                    post_data: e.detail.value,
                    data_page: 1
                });
                this.popup_form_event_close();
                this.get_data_list(1);
            },

            // 筛选条件关闭
            popup_form_event_close(e) {
                this.setData({
                    is_show_popup_form: false
                });
            },

            // 筛选条件开启
            popup_form_event_show(e) {
                this.setData({
                    is_show_popup_form: true
                });
            },

            // 排序事件
            nav_sort_event(e) {
                var index = e.currentTarget.dataset.index || 0;
                var temp_post_data = this.post_data;
                var temp_search_nav_sort = this.search_nav_sort_list;
                var temp_sort = temp_search_nav_sort[index]['sort'] == 'desc' ? 'asc' : 'desc';
                for (var i in temp_search_nav_sort) {
                    if (i != index) {
                        if (temp_search_nav_sort[i]['icon'] != null) {
                            temp_search_nav_sort[i]['icon'] = 'default';
                        }
                        temp_search_nav_sort[i]['sort'] = 'desc';
                    }
                }
                temp_search_nav_sort[index]['sort'] = temp_sort;
                if (temp_search_nav_sort[index]['icon'] != null) {
                    temp_search_nav_sort[index]['icon'] = temp_sort;
                }
                temp_post_data['order_by_field'] = temp_search_nav_sort[index]['field'];
                temp_post_data['order_by_type'] = temp_sort;
                this.setData({
                    post_data: temp_post_data,
                    search_nav_sort_list: temp_search_nav_sort,
                    data_page: 1
                });
                this.get_data_list(1);
            },

            // 条件-更多数据展示事件
            more_event(e) {
                var value = e.currentTarget.dataset.value || null;
                var temp_more = this.map_fields_list;
                if (value != null && (temp_more[value] || null) != null) {
                    temp_more[value]['height'] = temp_more[value]['height'] == 'auto' ? temp_more[value]['default'] : 'auto';
                    this.setData({
                        map_fields_list: temp_more
                    });
                }
            },

            // 条件-选择事件
            map_item_event(e) {
                var index = e.currentTarget.dataset.index;
                var field = e.currentTarget.dataset.field;
                var temp_list = this.search_map_list;                
                if ((temp_list[field] || null) != null && (temp_list[field][index] || null) != null) {
                    temp_list[field][index]['active'] = (temp_list[field][index]['active'] || 0) == 0 ? 1 : 0;
                    this.setData({
                        search_map_list: temp_list
                    });
                }
            },

            // 条件-清空
            map_remove_event(e) {
                var temp_list = this.search_map_list;
                var temp_post = this.post_data;
                
                // 关键字
                temp_post['wd'] = '';
                
                // 分类
                for (var i in temp_list) {
                    if((temp_list[i] || null) != null && temp_list[i].length > 0) {
                        for(var k in temp_list[i]) {
                            temp_list[i][k]['active'] = 0;
                        }
                    }
                }
                
                // 关闭弹窗、分页恢复1页、重新获取数据
                this.setData({
                    search_map_list: temp_list,
                    post_data: temp_post,
                    is_show_popup_form: false,
                    data_page: 1
                });
                this.get_data_list(1);
            }
        }
    };
</script>
<style>
    @import './search.css';
</style>