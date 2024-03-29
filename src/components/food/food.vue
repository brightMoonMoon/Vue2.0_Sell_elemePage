<template>
    <transition name="move">
        <div v-show="showFlag" class="food" ref="food">
            <div class="food-content">
                <div class="image-header">
                    <img :src="food.image" alt="食物图片">
                    <div class="back" @click="back">
                        <i class="iconfont icon-xia"></i>
                    </div>
                </div>
                <div class="content">
                    <h1 class="title">{{food.name}}</h1>
                    <div class="detail">
                        <span class="sell-count">月售{{food.sellCount}}</span>
                        <span class="rating">好评率{{food.rating}}%</span>
                    </div>
                    <div class="price">
                        <span class="now">￥{{food.price}}</span>
                        <span class="old" v-show="food.oldPrice">￥{{food.oldPrice}}</span>
                    </div>
                    <div class="cartcontrol-wrapper">
                        <v-cartcontrol :food="food"></v-cartcontrol>
                    </div>
                    <transition name="fade">
                        <div class="buy" @click="addFirst($event)" v-show="!food.count || food.count === 0">加入购物车</div>
                    </transition>
                </div>
                <v-split v-show="food.info"></v-split>
                <div class="info" v-show="food.info">
                    <h1 class="title">商品信息</h1>
                    <p class="text">{{food.info}}</p>
                </div>
                <v-split></v-split>
                <div class="rating">
                    <h1 class="title">商品评价</h1>
                    <div class="rating-select">
                        <v-ratingselect :select-type="selectType" :only-content="onlyContent" :desc="desc" :ratings="food.ratings"></v-ratingselect>
                    </div>
                    <div class="rating-wrapper">
                        <ul v-show="food.ratings && food.ratings.length">
                            <li v-show="needShow(rating.rateType,rating.text)" v-for="(rating,index) in food.ratings" class="rating-item" :key="index">
                                <div class="user">
                                    <span class="name">{{rating.username}}</span>
                                    <img class="avatar" :src="rating.avatar" alt="img" width="12px" height="12px">
                                </div>
                                <div class="time">{{rating.rateTime|formatDate}}</div>
                                <p class="text">
                                    <i class="iconfont" :class="{'icon-fandui':rating.rateType == 1,'icon-dianzan':rating.rateType == 0}"></i>
                                    {{rating.text}}
                                </p>
                            </li>
                        </ul>
                        <div class="no-rating" v-show="!food.ratings || food.ratings.length">
                            暂无评价
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </transition>
</template>


<script>
import BScroll from 'better-scroll';
import cartcontrol from '@/components/cartcontrol/cartcontrol';
import split from '@/components/split/split';
import ratingselect from '@/components/ratingselect/ratingselect';
import { formatDate } from '../../common/js/date.js'
const POSITIVE = 0;
const NEGATIVE = 1;
const ALL = 2;
export default {
    props: {
        food: {
            type: Object
        },
    },
    data() {
        return {
            showFlag: false,
            selectType: ALL,
            onlyContent: true,
            desc: {
                all: '全部',
                positive: '推荐',
                negative: '吐槽'
            },
        }
    },
    created() {
        this.$root.$on('ratingtype.select', (type) => {
            this.selectType = type;
            this.$nextTick(() => {
                // 数据改变后，dom更新，因为是异步的，所以得重新计算下高度
                this.scroll.refresh();
            })
        });
        this.$root.$on('content.toggle', (oBool) => {
            this.onlyContent = oBool;
            this.$nextTick(() => {
                this.scroll.refresh();
            })
        });
    },
    methods: {
        show() {
            this.selecType = ALL;
            this.onlyContent = true;
            this.showFlag = true;
            this.$nextTick(() => {
                if (!this.scroll) {
                    this.scroll = new BScroll(this.$refs.food, {
                        click: true
                    });
                } else {
                    this.scroll.refresh();
                }
            });
        },
        back() {
            this.showFlag = false
        },
        addFirst(event) {
            this.$root.$emit('cart-add', event.target)
            this.$set(this.food, 'count', 1);

        },
        needShow(type, text) {
            if (this.onlyContent && !text) {
                return false;
            }
            if (this.selectType === ALL) {
                return true;
            }
            else {
                return type === this.selectType;
            }
        }

    },
    filters: {
        formatDate(time) {
            let date = new Date(time);
            return formatDate(date, 'yyy-MM-dd hh:mm')
        }
    },
    components: {
        'v-cartcontrol': cartcontrol,
        'v-split': split,
        'v-ratingselect': ratingselect
    }
}
</script>


<style lang='scss' scoped>
@import '../../common/sass/index';
.food {
    position: fixed;
    left: 0;
    top: 0;
    bottom: 48px;
    overflow: hidden;
    z-index: 8;
    width: 100%;
    background: #fff;
    &.move-enter-active,
    &.move-leave-active {
        transition: all 0.3s linear;
    }
    &.move-enter,
    &.move-leave-to {
        transform: translate3d(100%, 0, 0)
    }
    .food-content {
        position: relative;
        .image-header {
            position: relative;
            width: 100%;
            height: 0;
            padding-top: 100%;
            img {
                position: absolute;
                top: 0;
                left: 0;
                width: 100%;
                height: 100%;
            }
            .back {
                position: absolute;
                top: 10px;
                left: 0;
                .icon-xia {
                    display: block;
                    padding: 10px;
                    font-size: 20px;
                    color: #fff;
                }
            }
        }
        .content {
            position: relative;
            padding: 18px;
            .title {
                line-height: 14px;
                margin-bottom: 8px;
                font-size: 14px;
                font-weight: 700;
                color: rgb(7, 17, 27);
            }
            .detail {
                margin-bottom: 18px;
                line-height: 10px;
                font-size: 0;
                .sell-count,
                .rating {
                    font-size: 10px;
                    color: rgb(147, 153, 159);
                }
                .sell-count {
                    margin-right: 12px;
                }
            }
            .price {
                font-weight: 700;
                line-height: 24px;
                .now {
                    margin-right: 8px;
                    font-size: 14px;
                    color: rgb(240, 20, 20);
                }
                .old {
                    text-decoration: line-through;
                    font-size: 10px;
                    color: rgb(147, 153, 159);
                }
            }
            .cartcontrol-wrapper {
                position: absolute;
                right: 12px;
                bottom: 12px; // 为什么写12px就成，是因为本身上下左右就有px的padding值
            }
            .buy {
                position: absolute;
                right: 18px;
                bottom: 18px;
                z-index: 10;
                height: 24px;
                line-height: 24px;
                padding: 0 12px; // 不写宽度能撑起来是因为有padding     
                box-sizing: border-box;
                border-radius: 12px;
                font-size: 10px;
                color: #fff;
                background: rgb(0, 160, 220);
                &.fade-enter-active,
                &.fade-leave-active {
                    transition: all 0.5s;
                }
                &.fade-enter,
                &.fade-leave-to {
                    opacity: 0;
                }
            }
        }
    }
    .info {
        padding: 18px;
        .title {
            line-height: 14px;
            margin-bottom: 6px;
            font-size: 14px;
            color: rgb(7, 17, 27);
        }
        .text {
            line-height: 24px;
            padding: 0 8px;
            font-size: 12px;
            color: rgb(77, 85, 93)
        }
    }
    .rating {
        padding-top: 18px;
        .title {
            line-height: 14px;
            margin-left: 6px;
            font-size: 14px;
            color: rgb(7, 17, 27)
        }
        .rating-wrapper {
            padding: 0 18px;
            .rating-item {
                position: relative;
                padding: 16px 0;
                @include border-1px(rgba(7, 17, 27, 0.1));
            }
            .user {
                position: absolute;
                right: 0;
                top: 16px;
                line-height: 12px;
                font-size: 0;
                .name {
                    display: inline-block;
                    margin-right: 6px;
                    vertical-align: top;
                    font-size: 10px;
                    color: rgb(147, 153, 159);
                }
                .avatar {
                    border-radius: 50%;
                }
            }
            .time {
                margin-bottom: 6px;
                line-height: 12px;
                font-size: 10px;
                color: rgb(147, 153, 159);
            }
            .text {
                line-height: 16px;
                font-size: 12px;
                color: rgb(0, 0, 0);
                .icon-fandui,
                .icon-dianzan {
                    margin-right: 4px;
                    line-height: 16px;
                    font-size: 12px;
                }
                .icon-fandui {
                    color: rgb(147, 153, 159)
                }
                .icon-dianzan {
                    color: rgb(0, 160, 220);
                }
            }
        }
    }
    .no-rating {
        padding: 16px 0;
        font-size: 12x;
        color: rgb(147, 153, 159)
    }
}
</style>