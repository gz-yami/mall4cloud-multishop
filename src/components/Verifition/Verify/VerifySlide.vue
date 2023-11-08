<template>
    <div style="position: relative;">
        <div v-if="type === '2'" class="verify-img-out"
             :style="{height: (parseInt(setSize.imgHeight) + vSpace) + 'px'}"
            >
            <div class="verify-img-panel" :style="{width: setSize.imgWidth,
                                                   height: setSize.imgHeight,}">
                <img :src="'data:image/png;base64,'+backImgBase" alt="" style="width:100%;height:100%;display:block">
                <div class="verify-refresh" @click="refresh" v-show="showRefresh"><i class="iconfont icon-refresh"></i>
                </div>
                <transition name="tips">
                    <span class="verify-tips" v-if="tipWords" :class="passFlag ?'suc-bg':'err-bg'">{{tipWords}}</span>
                </transition>
            </div>
        </div>
        <!-- 公共部分 -->
        <div class="verify-bar-area" :style="{width: setSize.imgWidth,
                                              height: barSize.height,
                                              'line-height':barSize.height}">
            <span class="verify-msg" v-text="text"></span>
            <div class="verify-left-bar"
                 :style="{width: (leftBarWidth!==undefined)?leftBarWidth: barSize.height, height: barSize.height, 'border-color': leftBarBorderColor, transaction: transitionWidth}">
                <span class="verify-msg" v-text="finishText"></span>
                <div class="verify-move-block"
                     @touchstart="start"
                     @mousedown="start"
                     :style="{width: barSize.height, height: barSize.height, 'background-color': moveBlockBackgroundColor, left: moveBlockLeft, transition: transitionLeft}">
                    <i :class="['verify-icon iconfont', iconClass]"
                       :style="{color: iconColor}"></i>
                    <div v-if="type === '2'"
                         class="verify-sub-block"
                         :style="{'width':Math.floor(parseInt(setSize.imgWidth)*47/310)+ 'px',
                                  'height': setSize.imgHeight,
                                  'top':'-' + (parseInt(setSize.imgHeight) + vSpace) + 'px',
                                  'background-size': setSize.imgWidth + ' ' + setSize.imgHeight,
                                  }">
                         <img :src="'data:image/png;base64,'+blockBackImgBase" alt=""  style="width:100%;height:100%;display:block">
                         </div>
                </div>
            </div>
        </div>
    </div>
</template>
<script type="text/babel">
    /**
     * VerifySlide
     * @description 滑块
     * */
    import {aesEncrypt} from "./../utils/ase"
    import {resetSize} from './../utils/util'
    import {reqGet,reqCheck}  from "./../api/index"

    //  "captchaType":"blockPuzzle",
    
      
        props: {
            captchaType:{
                type:String,
            },
            type: {
                type: String,
                default: '1'
            },
            //弹出式pop，固定fixed
            mode: {
                type: String,
                default: 'fixed'
            },
            vSpace: {
                type: Number,
                default: 5
            },
            explain: {
                type: String,
                default: '向右滑动完成验证'
            },
            imgSize: {
                type: Object,
                default() {
                    return {
                        width: '310px',
                        height: '155px'
                    }
                }
            },
            blockSize: {
                type: Object,
                default() {
                    return {
                        width: '50px',
                        height: '50px'
                    }
                }
            },
            barSize: {
                type: Object,
                default() {
                    return {
                        width: '310px',
                        height: '40px'
                    }
                }
            }
        },
        data () {
            return {
                secretKey:'',        //后端返回的加密秘钥 字段
                passFlag:'',         //是否通过的标识
                backImgBase:'',      //验证码背景图片
                blockBackImgBase:'', //验证滑块的背景图片
                backToken:"",        //后端返回的唯一token值
                startMoveTime:"",    //移动开始的时间
                endMovetime:'',      //移动结束的时间
                tipsBackColor:'',    //提示词的背景颜色
                tipWords:'',
                text: '',
                finishText:'',
                setSize: {
                    imgHeight: 0,
                    imgWidth: 0,
                    barHeight: 0,
                    barWidth: 0
                },
                top: 0,
                left: 0,
                moveBlockLeft: undefined,
                leftBarWidth: undefined,
                // 移动中样式
                moveBlockBackgroundColor: undefined,
                leftBarBorderColor: '#ddd',
                iconColor: undefined,
                iconClass: 'icon-right',
                status: false,      //鼠标状态
                isEnd: false,    //是够验证完成
                showRefresh: true,
                transitionLeft: '',
                transitionWidth: ''
            }
        },
        computed: {
            barArea() {
                return $el.querySelector('.verify-bar-area')
            },
            resetSize() {
                return resetSize
            }
        },
        methods: {
            init() {
                text = explain
                getPictrue();
                nextTick(() => {
                    let setSize = resetSize(this)  //重新设置宽度高度
                    for (let key in setSize) {
                        setSize[key] = setSize[key]
                    }
                    $parent.$emit('ready', this)
                })

                var _this = this

                window.removeEventListener("touchmove", function (e) {
                    _move(e);
                });
                window.removeEventListener("mousemove", function (e) {
                    _move(e);
                });

                //鼠标松开
                window.removeEventListener("touchend", function () {
                    _end();
                });
                window.removeEventListener("mouseup", function () {
                    _end();
                });

                window.addEventListener("touchmove", function (e) {
                    _move(e);
                });
                window.addEventListener("mousemove", function (e) {
                    _move(e);
                });

                //鼠标松开
                window.addEventListener("touchend", function () {
                    _end();
                });
                window.addEventListener("mouseup", function () {
                    _end();
                });
            },

            //鼠标按下
            start: function (e) {
                e = e || window.event
                if (!e.touches) {  //兼容PC端 
                    var x = e.clientX;
                } else {           //兼容移动端
                    var x = e.touches[0].pageX;
                }
                startLeft =Math.floor(x - barArea.getBoundingClientRect().left);
                startMoveTime = +new Date();    //开始滑动的时间
                if (isEnd == false) {
                    text = ''
                    moveBlockBackgroundColor = '#337ab7'
                    leftBarBorderColor = '#337AB7'
                    iconColor = '#fff'
                    e.stopPropagation();
                    status = true;
                }
            },
            //鼠标移动
            move: function (e) {
                e = e || window.event
                if (status && isEnd == false) {
                    if (!e.touches) {  //兼容PC端 
                        var x = e.clientX;
                    } else {           //兼容移动端
                        var x = e.touches[0].pageX;
                    }
                    var bar_area_left = barArea.getBoundingClientRect().left;
                    var move_block_left = x - bar_area_left //小方块相对于父元素的left值
                    if (move_block_left >= barArea.offsetWidth - parseInt(parseInt(blockSize.width) / 2) - 2) {
                        move_block_left = barArea.offsetWidth - parseInt(parseInt(blockSize.width) / 2) - 2;
                    }
                    if (move_block_left <= 0) {
                        move_block_left = parseInt(parseInt(blockSize.width) / 2);
                    }
                    //拖动后小方块的left值
                    moveBlockLeft = (move_block_left - startLeft) + "px"
                    leftBarWidth = (move_block_left - startLeft) + "px"
                }
            },

            //鼠标松开
            end: function () {
                endMovetime = +new Date(); 
                var _this = this;
                //判断是否重合
                if (status && isEnd == false) {
                    var moveLeftDistance = parseInt((moveBlockLeft || '').replace('px', ''));
                    moveLeftDistance = moveLeftDistance * 310/ parseInt(setSize.imgWidth)
                    let data = {
                        captchaType:captchaType,
                        "pointJson":secretKey ? aesEncrypt(JSON.stringify({x:moveLeftDistance,y:5.0}),secretKey):JSON.stringify({x:moveLeftDistance,y:5.0}),
                        "token":backToken
                    }
                    reqCheck(data).then(res=>{
                        if (res.repCode == "0000") {
                            moveBlockBackgroundColor = '#5cb85c'
                            leftBarBorderColor = '#5cb85c'
                            iconColor = '#fff'
                            iconClass = 'icon-check'
                            showRefresh = false
                            isEnd = true;   
                            if (mode=='pop') {
                                setTimeout(()=>{
                                    $parent.clickShow = false;
                                    refresh();
                                },1500)
                            }
                            passFlag = true
                            tipWords = `${((endMovetime-startMoveTime)/1000).toFixed(2)}s验证成功`
                            var captchaVerification = secretKey ? aesEncrypt(backToken+'---'+JSON.stringify({x:moveLeftDistance,y:5.0}),secretKey):backToken+'---'+JSON.stringify({x:moveLeftDistance,y:5.0})
                            setTimeout(()=>{
                                tipWords = ""
                                $parent.closeBox();
                                $parent.$emit('success', {captchaVerification})
                            },1000)
                        }else{
                            moveBlockBackgroundColor = '#d9534f'
                            leftBarBorderColor = '#d9534f'
                            iconColor = '#fff'
                            iconClass = 'icon-close'
                            passFlag = false
                            setTimeout(function () {
                                _refresh();
                            }, 1000);
                            $parent.$emit('error',this)
                            tipWords = "验证失败"
                            setTimeout(()=>{
                                    tipWords = ""
                            },1000)
                        }
                    })
                    status = false;
                }
            },

            refresh: function () {
                showRefresh = true
                finishText = ''

                transitionLeft = 'left .3s'
                moveBlockLeft = 0

                leftBarWidth = undefined
                transitionWidth = 'width .3s'

                leftBarBorderColor = '#ddd'
                moveBlockBackgroundColor = '#fff'
                iconColor = '#000'
                iconClass = 'icon-right'
                isEnd = false

                getPictrue()
                setTimeout(() => {
                    transitionWidth = ''
                    transitionLeft = ''
                    text = explain
                }, 300)
            },

            // 请求背景图片和验证图片
            getPictrue(){
                let data = {
                    captchaType:captchaType
                }
                reqGet(data).then(res=>{
                    if (res.repCode == "0000") {
                        backImgBase = res.repData.originalImageBase64
                        blockBackImgBase = res.repData.jigsawImageBase64
                        backToken = res.repData.token
                        secretKey = res.repData.secretKey
                    }else{
                        tipWords = res.repMsg;
                    }
                })
            },
        },
        watch: {
            // type变化则全面刷新
            type: {
                immediate: true,
                handler() {
                    init()
                }
            }
        },
        mounted() {
            // 禁止拖拽
            $el.onselectstart = function () {
                return false
            }
        },
    
</script>
