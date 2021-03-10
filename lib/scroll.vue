<template>
    <div class="scroll-container">
        <div class="scroll-box" ref="scrollBox" @scroll="onscroll" @wheel="onwheel">
            <div class="scroll-content" ref="scrollContent">
                <slot></slot>
            </div>
        </div>
        <div v-if="showScrollBarX" class="scroll-bar-x" ref="scrollBarX" :style="{height: scrollObj.offsetY + 'px'}">
            <div class="scroll-bar-x-inner"
                 ref="scrollBarXInner"
                 :style="scrollBarStyleX"
                 @mouseenter="scrollBarStyleX.background = initData.hoverColor"
                 @mouseleave="scrollBarStyleX.background = initData.color"
            ></div>
        </div>
        <div v-if="showScrollBarY" class="scroll-bar-y" ref="scrollBarY" style="border-radius: 3px" :style="{width: scrollObj.offsetX + 'px'}">
            <div class="scroll-bar-y-inner"
                 ref="scrollBarYInner"
                 :style="scrollBarStyleY"
                 @mouseenter="scrollBarStyleY.background = initData.hoverColor"
                 @mouseleave="scrollBarStyleY.background = initData.color"
            ></div>
        </div>
    </div>
</template>

<script>
    export default {
        name: "scroll",
        props: {
            /*默认初始化数据
                {
                  width: 8, 滚动条宽度
                  color: '#a9a9a9', 滚动条背景色
                  hoverColor: '#817878', 滚动条 激活时的颜色
                }
            */
            initData: {
                type: Object,
                default() {
                    return {
                        width: 8,
                        color: '#a9a9a9',
                        hoverColor: '#817878'
                    }
                }
            }
        },
        data() {
            return {
                scrollObj: {
                    xRadix: 0,
                    yRadix: 0,
                    offsetX: 0,
                    offSetY: 0
                },
                scrollBarStyleX: {
                    width: 0,
                    height: '6px',
                    borderRadius: '3px',
                    marginTop: '-3px',
                    background: '',
                    left: 0
                },
                scrollBarStyleY: {
                    width: '6px',
                    height: 0,
                    borderRadius: '3px',
                    marginLeft: '-3px',
                    background: '',
                    top: 0
                },
                direction: 'vertical', // 鼠标滚轮 控制 滚动条方向
                showScrollBarX: false, // 是否显示 横向滚动条
                showScrollBarY: false, // 是否显示纵向滚动条
                flag: true, // 触发底部事件状态
                changeStatus: false // 内容区是否变化
            }
        },
        methods: {
            // 初始化
            init() {
                const {scrollBox,scrollContent} = this.$refs;
                this.scrollObj = {
                    xRadix: scrollBox.clientWidth/scrollContent.clientWidth,
                    yRadix: scrollBox.clientHeight/scrollContent.clientHeight,
                    offsetX: scrollBox.offsetWidth - scrollBox.clientWidth || 17,
                    offsetY: scrollBox.offsetHeight - scrollBox.clientHeight || 17
                };
                this.initData.width > 17 ? this.initData.width = 17: this.initData.width;
                this.scrollBarStyleX = {
                    width: scrollBox.clientWidth * this.scrollObj.xRadix + 'px',
                    height: this.initData.width + 'px' || '6px',
                    borderRadius: this.initData.width/2 + 'px' || '3px',
                    marginTop: -this.initData.width/2 + 'px' || '-3px',
                    background: this.initData.color || '#a9a9a9',
                    left: scrollBox.scrollLeft > 0 ?scrollBox.scrollLeft* this.scrollObj.xRadix - this.scrollObj.offsetY + 'px': this.scrollObj.offsetY + 'px'
                }
                this.scrollBarStyleY = {
                    width: this.initData.width + 'px' || '6px',
                    borderRadius: this.initData.width/2 + 'px' || '3px',
                    marginLeft: -this.initData.width/2 + 'px' || '-3px',
                    height: scrollBox.clientHeight * this.scrollObj.yRadix + 'px',
                    background: this.initData.color || '#a9a9a9',
                    top: scrollBox.scrollTop > 0 ?scrollBox.scrollTop* this.scrollObj.yRadix - this.scrollObj.offsetX + 'px': this.scrollObj.offsetX + 'px'
                }
                if (scrollContent.offsetWidth > scrollBox.offsetWidth) {
                    this.showScrollBarX = true;
                    this.$nextTick(function () {
                        this.$refs.scrollBarXInner.onmousedown = this.onmousedownX
                    })
                } else {
                    this.showScrollBarX = false;
                }
                if (scrollContent.offsetHeight > scrollBox.offsetHeight) {
                    this.showScrollBarY = true;
                    this.$nextTick(function () {
                        this.$refs.scrollBarYInner.onmousedown = this.onmousedownY
                    })
                } else {
                    this.showScrollBarY = false;
                }
            },
            onwheel(e) {
                e.preventDefault();
                const {scrollBox,scrollContent} = this.$refs;
                if (scrollContent.offsetHeight <= scrollBox.offsetHeight) {
                    this.direction = 'horizontal';
                    if (e.deltaY > 0) {
                        scrollBox.scrollLeft += 100;
                    } else {
                        scrollBox.scrollLeft -= 100;
                    }
                } else {
                    this.direction = 'vertical';
                    if (!this.flag && this.changeStatus) {
                       return false
                    }
                    if (e.deltaY > 0) {
                        scrollBox.scrollTop += 100;
                    } else {
                        scrollBox.scrollTop -= 100;
                    }
                }

            },
            onmousedownY(_e) {
                this.direction = 'vertical';
                const {scrollBox,scrollBarY, scrollBarYInner} = this.$refs;
                _e = _e || window.event
                let _y = _e.clientY - scrollBarYInner.offsetTop;
                document.onmousemove = (e) => {
                    this.scrollBarStyleY.background = this.initData.hoverColor;
                    e = e || window.event;
                    let y = e.clientY - _y;
                    if (y < 0) {
                        y = 0;
                        this.scrollBarStyleY.top = y + this.scrollObj.offsetX + "px";
                    }
                    if (y > scrollBarY.offsetHeight - scrollBarYInner.offsetHeight) {
                        y = scrollBarY.offsetHeight - scrollBarYInner.offsetHeight;
                        this.scrollBarStyleY.top = y - this.scrollObj.offsetX + "px";
                    }
                    scrollBox.scrollTop = y/this.scrollObj.yRadix + this.scrollObj.offsetX;
                    if (y === 0) {
                        scrollBox.scrollTop = 0
                    }
                }
                document.onmouseup = () => {
                    this.scrollBarStyleY.background = this.initData.color;
                    document.onmousemove = null;
                    document.onmouseup = null
                }
                return false;
            },
            onmousedownX(_e) {
                this.direction = 'horizontal';
                const {scrollBox,scrollBarX, scrollBarXInner} = this.$refs;
                _e = _e || window.event
                let _x = _e.clientX - scrollBarXInner.offsetLeft;
                document.onmousemove = (e)=> {
                    this.scrollBarStyleX.background = this.initData.hoverColor;
                    e = e || window.event
                    let x = e.clientX - _x;
                    if (x < 0) {
                        x = 0;
                        this.scrollBarStyleX.left = x + this.scrollObj.offsetY + "px";
                    }
                    if (x > scrollBarX.offsetWidth - scrollBarXInner.offsetWidth) {
                        x = scrollBarX.offsetWidth - scrollBarXInner.offsetWidth;
                        this.scrollBarStyleX.left = x - this.scrollObj.offsetY + "px";
                    }
                    scrollBox.scrollLeft = x/this.scrollObj.xRadix + this.scrollObj.offsetY;
                    if (x === 0) {
                        scrollBox.scrollLeft = 0
                    }
                }
                document.onmouseup = () => {
                    this.scrollBarStyleX.background = this.initData.color;
                    document.onmousemove = null;
                    document.onmouseup = null;
                }
                return false;
            },
            onscroll(e) {
                e = e || window.event
                const target = e.target || this.$refs['scrollBox'];
                if (this.direction === 'vertical') {
                    if (target.scrollTop < this.scrollObj.offsetX / this.scrollObj.yRadix) {
                        if (target.scrollTop === 0) {
                            this.scrollTopEvent(e);
                        }
                        this.scrollBarStyleY.top = this.scrollObj.offsetX + 'px';
                    } else {
                        this.scrollBarStyleY.top = target.scrollTop* this.scrollObj.yRadix + 'px';
                        if (target.scrollTop === (target.scrollHeight - target.clientHeight)) {
                            if (this.showScrollBarX) {
                                this.scrollBarStyleY.top = target.scrollTop* this.scrollObj.yRadix + 'px';
                            } else {
                                this.scrollBarStyleY.top = target.scrollTop* this.scrollObj.yRadix - this.scrollObj.offsetX + 'px';
                            }
                            this.scrollBottomEvent(e);
                            this.$refs.scrollBox.onmousedown = null;
                            document.onmousemove = null;
                        }
                        if (!this.flag) {
                            return
                        }
                    }
                } else {
                    if (target.scrollLeft < this.scrollObj.offsetY / this.scrollObj.xRadix) {
                        this.scrollBarStyleX.left = this.scrollObj.offsetY + 'px';
                    } else {
                        this.scrollBarStyleX.left = target.scrollLeft* this.scrollObj.xRadix + 'px';
                        if (target.scrollLeft === (target.scrollWidth - target.clientWidth)) {
                            if (this.showScrollBarY) {
                                this.scrollBarStyleX.left = target.scrollLeft* this.scrollObj.xRadix + 'px';
                            } else {
                                this.scrollBarStyleX.left = target.scrollLeft* this.scrollObj.xRadix - this.scrollObj.offsetY + 'px';
                            }
                        }
                    }
                }
            },
            scrollBottomEvent(e) {
                if (this.flag) {
                    this.$emit('scrollBottom', e);
                    this.flag = false;
                }
            },
            scrollTopEvent(e) {
                this.$emit('scrollTop', e);
            },
            scrollBy(...args) {
                this.$refs.scrollBox.scrollBy(...args)
            },
            scrollTo(...args) {
                this.$refs.scrollBox.scrollTo(...args)
            },
            scroll(...args) {
                this.$refs.scrollBox.scroll(...args)
            }
        },
        mounted() {
            this.$nextTick(function () {
                this.init();
                let MutationObserver = window.MutationObserver || window.WebKitMutationObserver || window.MozMutationObserver
                // 选择需要观察变动的节点
                const targetNode = this.$refs.scrollContent;
                // 观察器的配置（需要观察什么变动）
                const config = {childList: true,characterData: true, attributes: true, attributeFilter: ['style', 'class'], attributeOldValue:true, subtree: true };
                // 当观察到变动时执行的回调函数
                const callback = (mutationsList, observer)=> {
                    this.changeStatus = true;
                    // Use traditional 'for loops' for IE 11
                    for(let mutation of mutationsList) {
                        setTimeout(() => {
                            this.init();
                            this.flag = true;
                        }, 500);
                    }
                };
                // 创建一个观察器实例并传入回调函数
                let observer = new MutationObserver(callback);

                // 以上述配置开始观察目标节点
                observer.observe(targetNode, config);
                window.onresize = (e) => {
                    this.init();
                }
            })
        }
    }
</script>

<style scoped>
    .scroll-container {
        height: 100%;
        width: 100%;
        position: relative;
        overflow: hidden;
    }
    .scroll-box {
        position: absolute; left: 0;
        overflow: auto;
        height: 100%;
        width: 100%;
    }
    .scroll-bar-x {
        position: absolute;
        bottom: 0;
        left: 0;
        width: 100%;
        background: #fff;
    }
    .scroll-bar-y {
        position: absolute;
        height: 100%;
        right: 0;
        top: 0;
        background: #fff;
    }
    .scroll-content {
        width: auto;
        height: auto;
        display: inline-block;
    }
    .scroll-bar-y-inner {
        height: 100%;
        background: #a9a9a9;
        position: absolute;
        left: 50%;
        cursor: pointer;
    }
    .scroll-bar-x-inner {
        width: 100%;
        background: #a9a9a9;
        position: absolute;
        top: 50%;
        cursor: pointer;
    }
</style>
