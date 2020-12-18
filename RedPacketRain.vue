<template>
    <div v-show="isEnd" id="RedEnvelopeRain">
        <!-- 背景音乐 -->
        <audio ref="audioBgc" controls="controls" loop="loop" style="display: none;">
            <source src="@/assets/audio/bgc.wav" type="audio/mp3" />
        </audio>
        <!-- 中奖音乐 -->
        <audio ref="winAudio" controls="controls" style="display: none;">
            <source src="@/assets/audio/win.mp3" type="audio/mp3" />
        </audio>

        <!-- 红包雨主体 -->
        <div :style="{opacity,zIndex}" class="red-envelope-rain-mask">
            <!-- 倒计时 -->
            <div class="countdown" v-if="secondMask">
                <div class="countdown-dialog">
                    <img class="img" src="@/assets/RedEnvelopeRain/countdown-dialog.png" />
                    <p class="second">{{second}}</p>
                </div>
            </div>
            <!-- 红包 -->
            <div class="red-envelope-box" v-show="!secondMask">
                <!-- 已抢红包名单列表 -->
                <div class="win-list-box">
                    <p class="win-list-title">
                        <img class="title-img" src="@/assets/RedEnvelopeRain/win-title-icon.png" />
                        <!-- <span class="title-text">已抢红包名单</span> -->
                        <span class="title-text">{{$t('redPacket.winListTitle')}}</span>
                    </p>
                    <div id="winListWrap" class="win-list-wrap">
                        <ul id="winList" class="win-list">
                            <li class="win-list-item" v-for="(item,index) in winList" :key="index">
                                <p class="win-username">{{item.userName}}</p>
                                <!-- <p class="win-amount">{{item.amount}}元现金红包</p> -->
                                <p
                                    class="win-amount"
                                >{{item.amount}} {{$t('redPacket.winListAmount')}}</p>
                            </li>
                        </ul>
                        <ul id="winList2" class="win-list"></ul>
                    </div>
                </div>

                <!-- 已抢红包总金额 -->
                <div class="total">
                    <img class="total-img" src="@/assets/RedEnvelopeRain/total.png" />
                    <!-- <p class="total-title">已抢红包总金额</p> -->
                    <p class="total-title">{{$t('redPacket.totalAmountTitle')}}</p>
                    <p ref="totalMoney" class="total-money">{{totalMoney | filtAmount}}</p>
                </div>

                <!-- 隐藏/关闭按钮 -->
                <div class="top-btns">
                    <!-- <div @click="hide" class="btn">隐藏活动</div>
                    <div @click="isShowConfirm = true" class="btn">关闭活动</div>-->
                    <div @click="hide" class="btn">{{$t('redPacket.hideBtn')}}</div>
                    <div @click="isShowConfirm = true" class="btn">{{$t('redPacket.closeBtn')}}</div>
                </div>

                <!-- 红包雨 -->
                <ul class="red-envelope-list" id="red_packet">
                    <template v-for="(item, index) in liParams">
                        <li
                            class="red-envelope-list-item"
                            :style="{ left: item.left, animationDuration: item.durTime, webkitAnimationDuration: item.durTime}"
                            :data-index="index"
                            @webkitAnimationEnd="removeDom"
                            @click="tap(item)"
                            :key="index"
                        >
                            <i
                                :style="{transform: item.transforms, webkitTransform: item.transforms}"
                                :class="['red-envelope',{'defaul':item.status==0},{'success':item.status==1},{'fail':item.status==2}]"
                            />
                        </li>
                    </template>
                </ul>
            </div>
        </div>

        <!-- VIP升级 -->
        <div class="red-envelope-rain-mask" :style="{opacity:vipopacity,zIndex:vipzIndex}" v-if="vipMask">
            <div class="countdown" v-if="vipMask">
                <div class="countdown-dialog">
                    <div class="upgrade-img">
                        <p class="title-vip">Chúc mừng bạn đã nâng cấp</p>
                        <p class="viplevel">{{showViplevel}}</p>
                        <p class="text-vip">Nâng cấp thành công</p>
                        <p class="text-vip text-vip2">Phần thưởng sẽ được gửi trong vòng 24 giờ</p>
                        <el-button
                            @click="vipMask=false"
                            class="btn-confirm"
                            type="primary"
                        >Tôi biết</el-button>
                    </div>
                </div>
            </div>
        </div>

        <!-- 点击参与红包雨图标 -->
        <div v-drag id="moveToBtn" class="show-red-envelope-rain" v-if="!opacity && isHide">
            <!-- <p @click="showRedEnvelopeRain" class="move-to-btn">点击参与</p> -->
            <p @click="showRedEnvelopeRain" class="move-to-btn">{{$t('redPacket.join')}}</p>
        </div>

        <!-- 确认关闭红包雨弹窗 -->
        <div v-if="isShowConfirm" class="confirm-dialog">
            <!-- <p class="confirm-dialog-title">提示</p> -->
            <p class="confirm-dialog-title">{{$t('redPacket.confirmTitle')}}</p>
            <div class="confirm-dialog-content">
                <!-- <p>确定要关闭活动吗？</p> -->
                <p>{{$t('redPacket.confirmText')}}</p>
            </div>
            <div class="btn-box">
                <!-- <div @click="isShowConfirm = false" class="btn">取消</div>
                <div @click="close" class="btn">确认</div>-->
                <div @click="isShowConfirm = false" class="btn">{{$t('redPacket.cancelBtn')}}</div>
                <div @click="close" class="btn">{{$t('redPacket.confirmBtn')}}</div>
            </div>
        </div>

        <!-- 抢到红包弹窗 -->
        <transition name="win">
            <div v-if="isShowWin" class="win-dialog">
                <!-- <p class="title">恭喜你获得</p> -->
                <p class="title">{{$t('redPacket.winTitle')}}</p>
                <!-- <p class="name">现金红包</p> -->
                <p class="name">{{$t('redPacket.winText')}}</p>
                <p class="money">{{winMoney | filtAmount}}</p>
                <!-- <div @click="winMoney = 0" class="btn">关闭界面</div> -->
                <div @click="isShowWin = false" class="btn">{{$t('redPacket.winCloseBtn')}}</div>
            </div>
        </transition>
    </div>
</template>

<script>
import createWebSocket from "@/http/socket";
export default {
    watch: {
        "$store.state.islogin": function(newValue) {
            if (newValue) {
                this.initSocket();
            } else {
                this.initData();
                if (this.socket) this.socket.close();
            }
        }
    },
    filters: {
        // 过滤千位符 - 使用正则替换，每隔三个数加一个',
        filtAmount: (value, type = "amount") => {
            if (!value) return 0;
            if (type == "number") {
                return value.toString().replace(/(\d)(?=(\d{3})+$)/g, "$1,");
            } else {
                return value.toFixed(2).replace(/(\d)(?=(\d{3})+\.)/g, "$1,");
            }
        }
    },
    directives: {
        drag: {
            bind: function(el) {
                //监听document是因为如果监听元素el的话鼠标太快移出元素后就会失效
                el.onmousedown = event => {
                    //算出鼠标相对元素的位置
                    let pointX = event.clientX - el.offsetLeft; //鼠标位置X减去元素距离左边距离（鼠标到元素左边的距离）
                    let pointY = event.clientY - el.offsetTop; //鼠标位置Y减去距离顶部距离（鼠标到元素顶部的高度）

                    //解决拖动会选中文字的问题
                    document.onselectstart = function() {
                        return false;
                    };

                    document.onmousemove = e => {
                        //用鼠标的位置减去鼠标相对元素的位置，得到元素的位置
                        let left = e.clientX - pointX;
                        let top = e.clientY - pointY;
                        //移动当前元素
                        el.style.left = left + "px";
                        el.style.top = top + "px";
                    };

                    document.onmouseup = e => {
                        el.style.transition = "all 0.1s";
                        document.onmousemove = null;
                        document.onmouseup = null;
                    };
                };
            }
        }
    },

    data() {
        return {
            opacity: 0,
            isHide: false,
            zIndex: -999,

            socket: null, // socket 实例
            totalAmount: 0, // 本场红包总金额
            totalMoney: 0, // 已经抢红包总金额
            activeId: null, // 红包活动ID

            second: 5, // 红包雨开始倒计时
            secondMask: true, // 倒计时弹层开关
            duration: 1, // 红包雨持续时间/分钟

            liParams: [], // 渲染的红包数据
            list: [], // 后端返回的红包数据
            redPacketIndex: 0, // 添加渲染的红包索引
            isEnd: true, // 本场红包雨是否结束

            winList: [], // 中奖名单数据
            winMoney: 0, // 抢到的红包金额
            isShowWin: false,
            isShowConfirm: false, // 关闭红包雨确认开关

            timerData: {
                start: null, // 红包雨开场倒计时定时器
                end: null, // 红包雨结束计时定时器
                upTimer: null, // 中奖名单滚动定时器
                totalAmount: null, // 总金额增加的定时器
                redPacket: null // 每个红包的定时器
            },

            vipMask: true, //Vip升级弹窗
            vipopacity: 0, //Vip升级弹窗
            vipzIndex: -999, //Vip升级弹窗
            showViplevel: ""
        };
    },
    created() {
        if (this.$store.state.islogin) {
            this.initSocket();
        }
    },
    methods: {
        // 数据初始化
        initData() {
            //VIP升级
            this.vipzIndex = -999;
            this.vipopacity = 0;

            this.opacity = 0;
            this.isHide = false;
            this.zIndex = -999;
            this.totalAmount = 0; // 本场红包总金额
            this.totalMoney = 0; // 已经抢红包总金额
            this.activeId = null; // 红包活动ID
            this.second = 5; // 红包雨开始倒计时
            this.secondMask = true; // 倒计时弹层开关
            this.duration = 1; // 红包雨持续时间/分钟
            this.liParams = []; // 渲染的红包数据
            this.list = []; // 后端返回的红包数据
            this.redPacketIndex = 0; // 添加渲染的红包索引
            this.isEnd = true; // 本场红包雨是否结束
            this.winList = []; // 中奖名单数据
            this.winMoney = 0; // 抢到的红包金额
            this.isShowWin = false;
            this.isShowConfirm = false; // 关闭红包雨确认开关
            // 清除所有的定时器
            for (const key in this.timerData) {
                if (this.timerData[key]) {
                    clearInterval(this.timerData[key]);
                    this.timerData[key] = null;
                }
            }
        },
        // socket 连接
        initSocket() {
            let userName = sessionStorage.getItem("username") || null;
            // let socketUrl = `ws://192.168.1.97:8087/redPacketSocket/${userName}`
            let socketUrl = `wss://${this.$socket}/redPacketSocket/${userName}`;
            this.socket = createWebSocket({
                socketUrl: socketUrl,
                username: userName,
                heatBeat: {
                    event: "deviceHeatBeat",
                    data: userName
                },
                getMessage: res => {
                    if (res.code == 0 &&res.data.data.list &&res.data.data.list.length) {
                        this.totalAmount = res.data.data.data.totalAmount;
                        this.duration = res.data.data.data.aliveTime;
                        this.activeId = res.data.data.data.activeId;
                        this.list = res.data.data.list;
                        this.show();
                        this.payBgcAudio();
                    }
                    if (res.code == 9999) {
                        this.vipopacity = 1;
                        this.vipzIndex = 999;
                        this.showViplevel = JSON.parse(res.data).vipCode;
                    }
                }
            });
        },
        // 显示开启红包雨倒计时
        show() {
            this.opacity = 1;
            this.zIndex = 9999;
            this.countDownFn();
            this.getWinList();
        },
        // 红包雨开始倒计时
        countDownFn() {
            this.timerData.start = setInterval(() => {
                if (this.second == 1) {
                    this.secondMask = false;
                    clearInterval(this.timerData.start);
                    this.countDownEndFn();
                    this.initUpAnimation(50);
                    this.startRedPacket();
                } else {
                    this.second--;
                }
            }, 1000);
        },
        // 红包雨结束计时
        countDownEndFn() {
            let endTime = this.duration * 60000;
            this.timerData.end = setInterval(() => {
                if (endTime == 0) {
                    this.$refs.audioBgc.pause();
                    this.initData();
                } else {
                    endTime -= 1000;
                }
            }, 1000);
            // this.numberGrow();
        },
        // 总金额变动动画
        numberGrow() {
            let step = (this.duration * 60000) / (this.totalAmount / 7);
            let current = 0;
            let index = 0;
            this.timerData.totalAmount = setInterval(() => {
                index < this.winList.length - 1 ? index++ : (index = 0);
                if (this.winList) this.totalMoney + this.winList;
                this.totalMoney += this.winList[index].amount;
                if (!this.isEnd) clearInterval(this.timerData.totalAmount);
            }, 1000);
        },
        // 获取中奖名单数据
        getWinList() {
            this.$axios
                .post("/redPacket/getMemberReceiveList", {
                    activeId: this.activeId
                })
                .then(res => {
                    if (
                        res.data.code == 0 &&
                        res.data.data.list &&
                        res.data.data.list.length
                    ) {
                        this.winList = res.data.data.list;
                    }
                });
        },
        // 初始化中奖名单滚动效果
        initUpAnimation(t) {
            var box = document.getElementById("winListWrap");
            var ul1 = document.getElementById("winList");
            var ul2 = document.getElementById("winList2");
            ul2.innerHTML = ul1.innerHTML;
            box.scrollTop = 0;
            this.timerData.upTimer = setInterval(rollStart, t);

            function rollStart() {
                if (box.scrollTop >= ul1.scrollHeight) {
                    box.scrollTop = 0;
                } else {
                    box.scrollTop++;
                }
            }
        },

        // 初始化红包
        startRedPacket() {
            let windoWidth =
                document.documentElement.clientWidth ||
                document.body.clientWidth;
            let rotate = parseInt(Math.random() * 90 - 45) + "deg"; // 红包旋转角度
            let width = Math.random() * 10 + 100; // 红包随机宽度
            let height = Math.random() * 10 + 138; // 红包随机宽度
            let durTime = parseInt(Math.random() * 1.5) + 2.5 + "s"; // 红包飘过速度

            let left = parseInt(Math.random() * windoWidth);
            if (left <= 200) {
                left = windoWidth / 2;
            } else if (left > windoWidth - 200) {
                left = windoWidth / 2;
            }
            if (this.redPacketIndex >= this.list.length)
                this.redPacketIndex = 0;

            this.liParams.push({
                redId: this.list[this.redPacketIndex],
                left: left + "px", // 红包随机位置
                width: width + "px", // 红包随机宽度
                height: height + "px", // 红包随机高度
                transforms: "rotate(" + rotate + ")", // 红包旋转角度
                durTime: durTime, // 红包飘过速度
                status: 0 // 0 默认 1 中奖 2 未中奖
            });
            this.redPacketIndex++;

            // 多少时间结束
            setTimeout(() => {
                clearTimeout(this.timerData.redPacket);
                return false;
            }, this.duration * 60000);

            // 红包密度
            this.timerData.redPacket = setTimeout(() => {
                this.startRedPacket();
            }, 200);
        },
        // 移除红包元素
        removeDom(e) {
            let target = e.currentTarget;
            document.querySelector("#red_packet").removeChild(target);
        },
        // 点击红包
        tap(item) {
            if (item.status == 0) {
                let obj = { activeId: this.activeId, redId: item.redId };
                this.$axios
                    .post("/redPacket/receiveRedPackage", obj)
                    .then(res => {
                        if (res.status == 200 && res.data.data > 0) {
                            this.payWinAudio();
                            this.winMoney = res.data.data;
                            this.totalMoney += res.data.data;
                            this.isShowWin = true;
                            item.status = 1;
                        } else {
                            item.status = 2;
                        }
                    });
            }
        },

        // 隐藏红包雨
        hide() {
            this.opacity = 0;
            this.zIndex = -10;
            this.isHide = true;
            this.isShowWin = false;
            this.isShowConfirm = false;
        },
        // 点击参与/显示红包雨
        showRedEnvelopeRain() {
            this.opacity = 1;
            this.zIndex = 999;
        },
        // 关闭红包雨
        close() {
            this.$refs.audioBgc.pause();
            this.$refs.winAudio.pause();
            this.initData();
        },

        // 播放背景音乐
        payBgcAudio() {
            setTimeout(() => {
                let buttonAudio = this.$refs.audioBgc;
                buttonAudio.load();
                let playPromise = buttonAudio.play();
                if (playPromise !== undefined) {
                    playPromise
                        .then(() => {
                            buttonAudio.play();
                        })
                        .catch(() => {});
                }
            }, 1000);
        },
        // 播放中奖音乐
        payWinAudio() {
            let buttonAudio = this.$refs.winAudio;
            buttonAudio.load();
            let playPromise = buttonAudio.play();
            if (playPromise !== undefined) {
                playPromise
                    .then(() => {
                        buttonAudio.play();
                    })
                    .catch(() => {});
            }
        }
    },
    beforeDestroy() {
        this.initData();
        if (this.socket) this.socket.close();
    }
};
</script>

<style lang="less" scoped>
// 点击参与 -------------------------------------------------------------------------------------------
.show-red-envelope-rain {
    position: fixed;
    bottom: 100px;
    left: 50px;
    z-index: 9999;
    width: 202px;
    height: 255px;
    background: url("../../assets/RedEnvelopeRain/micrify-icon.png") no-repeat;
    background-size: 100% 100%;
    .move-to-btn {
        width: 80px;
        text-align: center;
        color: #a54206;
        font-size: 14px;
        font-weight: 700;
        position: absolute;
        bottom: 74px;
        left: 50%;
        transform: translateX(-50%);
        cursor: pointer;
        -webkit-user-select: none;
        -moz-user-select: none;
        -ms-user-select: none;
        user-select: none;
    }
}

// 红包雨遮罩层样式 -------------------------------------------------------------------------------------------
.red-envelope-rain-mask {
    position: fixed;
    left: 0;
    right: 0;
    top: 0;
    bottom: 0;
    // z-index: 999999;
    background-color: rgba(0, 0, 0, 0.5);
}

// 开场倒计时样式 ---------------------------------------------------------------------------------------------
.countdown {
    .countdown-dialog {
        position: absolute;
        top: 50%;
        left: 50%;
        z-index: 2;
        transform: translate(-50%, -50%);
    }
    .second {
        position: absolute;
        left: 44%;
        bottom: 100px;
        z-index: 2;
        font-size: 80px;
        font-weight: 700;
        color: #fff;
        text-shadow: 0px 0px 20px rgba(255, 255, 255, 1);
        // animation: mymove 1s infinite;
    }
    @keyframes mymove {
        0% {
            transform: scale(1); /*开始为原始大小*/
            opacity: 1;
        }
        100% {
            transform: scale(4);
            opacity: 0;
        }
    }
}

// 已抢红包名单列表 -------------------------------------------------------------------------------------------
.win-list-box {
    width: 460px;
    height: 330px;
    position: absolute;
    top: 60px;
    left: 40px;
    background-color: rgba(0, 0, 0, 0.5);
    border: 1px solid #a17a3b;
    padding: 20px 40px;
    display: flex;
    flex-direction: column;
    .win-list-title {
        display: flex;
        align-items: center;
        margin-bottom: 16px;
        .title-text {
            font-size: 22px;
            color: #ffd867;
            margin-left: 8px;
        }
    }
    .win-list-wrap {
        height: 240px;
        overflow: hidden;
        .win-list {
            .win-list-item {
                width: 378px;
                height: 24px;
                display: flex;
                align-items: center;
                justify-content: space-between;
                color: #fff;
                font-size: 14px;
            }
        }
    }
}

// 已抢红包总金额样式 -----------------------------------------------------------------------------------------
.total {
    width: 464px;
    height: 154px;
    margin: 0 auto;
    position: relative;
    -webkit-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
    user-select: none;
    .total-img {
        margin-top: 38px;
        -webkit-user-select: none;
        -moz-user-select: none;
        -ms-user-select: none;
        user-select: none;
    }
    .total-title {
        color: #fff;
        font-size: 14px;
        position: absolute;
        top: 53px;
        left: 50%;
        transform: translateX(-50%);
    }
    .total-money {
        color: #f83f28;
        font-size: 40px;
        font-weight: 700;
        position: absolute;
        bottom: 10px;
        left: 50%;
        transform: translateX(-50%);
    }
}

// 隐藏/关闭按钮 ----------------------------------------------------------------------------------------------
.top-btns {
    position: absolute;
    top: 100px;
    right: 40px;
    display: flex;
    .btn {
        padding: 10px;
        height: 40px;
        font-size: 18px;
        color: #fff;
        border: 1px solid #fff;
        border-radius: 6px;
        display: flex;
        align-items: center;
        justify-content: center;
        margin-right: 10px;
        cursor: pointer;
        text-transform: capitalize;
        -webkit-user-select: none;
        -moz-user-select: none;
        -ms-user-select: none;
        user-select: none;
        &:last-child {
            margin-right: 0px;
        }
        &:active {
            opacity: 0.9;
        }
    }
}

// 红包雨样式 -------------------------------------------------------------------------------------------------
.red-envelope-list {
    @keyframes aim_move {
        0% {
            transform: translateY(0);
        }
        100% {
            transform: translateY(120vh);
        }
    }
    .red-envelope-list-item {
        position: absolute;
        animation: all 3s linear;
        z-index: 3;
        animation: aim_move 5s linear 1 forwards;
        .red-envelope {
            width: 130px;
            height: 200px;
            display: block;
            -webkit-user-select: none;
            -moz-user-select: none;
            -ms-user-select: none;
            user-select: none;
            &.defaul {
                background: url("../../assets/RedEnvelopeRain/red-envelope.png")
                    no-repeat center;
                background-size: 100% 100%;
            }
            &.fail {
                background: url("../../assets/RedEnvelopeRain/red-envelope-empty.png")
                    no-repeat center;
                background-size: 100% 100%;
            }
            &.success {
                background: url("../../assets/RedEnvelopeRain/red-envelope-active.png")
                    no-repeat center;
                background-size: 100% 100%;
            }
        }
    }
}

// 确认关闭红包雨弹窗 -----------------------------------------------------------------------------------------
.confirm-dialog {
    width: 360px;
    background-color: rgba(0, 0, 0, 0.8);
    border: 1px solid rgba(255, 255, 255, 0.3);
    border-radius: 6px;
    position: fixed;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    z-index: 99999;
    padding: 40px 20px;
    font-size: 18px;
    color: #fff;
    .confirm-dialog-title {
        font-size: 24px;
        text-align: center;
    }
    .confirm-dialog-content {
        padding: 40px 0px;
        text-align: center;
    }
    .btn-box {
        display: flex;
        justify-content: center;
        .btn {
            width: 100px;
            height: 30px;
            font-size: 16px;
            color: #fff;
            border: 1px solid #fff;
            border-radius: 6px;
            display: flex;
            align-items: center;
            justify-content: center;
            margin-right: 10px;
            cursor: pointer;
            text-transform: capitalize;
            -webkit-user-select: none;
            -moz-user-select: none;
            -ms-user-select: none;
            user-select: none;
            &:last-child {
                margin-right: 0px;
            }
            &:active {
                opacity: 0.9;
            }
        }
    }
}

// 抢到红包弹窗 -----------------------------------------------------------------------------------------------
.win-dialog {
    width: 362px;
    height: 480px;
    background: url("../../assets/RedEnvelopeRain/win-dialog.png") no-repeat;
    background-size: 100% 100%;
    position: fixed;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    z-index: 9999;
    -webkit-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
    user-select: none;
    .title {
        width: 100%;
        text-align: center;
        font-size: 30px;
        font-weight: 700;
        color: #f83f28;
        position: absolute;
        top: 110px;
        left: 50%;
        transform: translateX(-50%);
    }
    .name {
        font-size: 20px;
        position: absolute;
        top: 160px;
        left: 50%;
        transform: translateX(-50%);
    }
    .money {
        font-size: 70px;
        font-weight: 700;
        color: #f83f28;
        position: absolute;
        top: 190px;
        left: 50%;
        transform: translateX(-50%);
    }
    .btn {
        width: 210px;
        text-align: center;
        font-size: 24px;
        position: absolute;
        bottom: 56px;
        left: 50%;
        transform: translateX(-50%);
    }
}

/* 定义进入过渡的开始状态 和 离开过渡的结束状态 */
.win-enter,
.win-leave-to {
    opacity: 0;
}
/* 定义进入和离开时候的过渡状态 */
.win-enter-active,
.win-leave-active {
    transition: all 0.3s ease;
}

.upgrade-img {
    background-image: url("../../assets/vip/vip-upgrade.png");
    width: 548px;
    height: 490px;
    position: relative;
    .title-vip {
        position: absolute;
        top: 130px;
        left: 50%;
        transform: translateX(-50%);
        font-size: 24px;
        color: #262626;
        width: 100%;
        text-align: center;
    }
    .viplevel {
        top: 160px;
        left: 50%;
        transform: translateX(-50%);
        position: absolute;
        font-size: 51px;
        font-family: Arial;
        font-weight: bold;
        color: rgba(38, 38, 38, 1);
        line-height: 93px;
    }
    .text-vip {
        top: 280px;
        left: 50%;
        color: #ce9e49;
        transform: translateX(-50%);
        position: absolute;
        font-size: 18px;
        font-family: Arial;
        font-weight: 400;
        line-height: 35px;
        width: 380px;
        text-align: center;
    }
    .text-vip2 {
        top: 320px;
    }
    .btn-confirm {
        font-size: 16px;
        background: #f7bd01;
        border: none;
        color: #262626;
        text-transform: uppercase;
        position: absolute;
        bottom: 70px;
        left: 50%;
        transform: translateX(-50%);
        width: 210px;
        margin: 0 auto;
    }
}
</style>