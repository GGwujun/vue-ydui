<template>
    <div class="yd-indexlist">
        <div class="yd-indexlist-tab" :style="{color: color, backgroundColor: bgcolor}" ref="navbox">
            <ul class="yd-indexlist-tab-item" ref="nav" :style="{color: currentColor, height: height}">
                <li :style="{color: color}" v-for="(item, i) in dataObj" :key="i" :class="activeIndex === i  ? 'yd-indexlist-current' : ''" :ref="'navitem_' + i" @click.stop="scrollContent(i)">
                    <span>{{item.key}}</span>
                </li>
            </ul>
        </div>
        <div class="yd-indexlist-content" ref="scrollView">
            <div :label="item.label" v-for="(item, key) in dataObj" :key="key" ref="scrollItem">
                <p class="indexlist-title">{{item.key}}</p>
                <yd-list :theme="6">
                    <yd-list-item v-for="i in item.value" :key="i.label">
                        <template>
                            <img slot="img" src="http://static.shikee.com/common/img/blank.gif" style="background-color:#e6e6e6;">
                            <span slot="title">
                                <span style="color:#909090;">{{i.name}}</span>
                            </span>
                        </template>
                    </yd-list-item>
                </yd-list>
            </div>
        </div>
    </div>
</template>

<script type="text/babel">
import { isColor, scrollTop, PinYin, trim } from '../../../utils/assist';

export default {
    name: 'yd-indexlist',
    data() {
        return {
            toggle: false,
            activeIndex: this.index,
            currentOffset: 0,
            currentPosition: 0,
            scrolling: false
        }
    },
    props: {
        indexname: {
            type: String,
            default: 'name'
        },
        index: {
            validator(val) {
                return /^(([1-9]\d*)|0)$/.test(val);
            },
            default: 0
        },
        height: {
            validator(value) {
                return /^(\.|\d+\.)?\d+(px|rem)$/.test(value);
            },
            // default: '.9rem'
        },
        color: {
            validator(value) {
                if (!value) return true;
                return isColor(value);
            },
            default: '#333'
        },
        currentColor: {
            validator(value) {
                if (!value) return true;
                return isColor(value);
            },
            default: '#F00'
        },
        toggleText: {
            type: String,
            default: '切换分类'
        },
        bgcolor: {
            validator(value) {
                if (!value) return true;
                return isColor(value);
            },
            default: '#FFF'
        },
        borderColor: {
            validator(value) {
                if (!value) return true;
                return isColor(value);
            },
            default: '#EFEFEF'
        },
        dataArray: {
            type: Array,
            default: function() {
                return []
            }
        },
        callback: {
            type: Function
        }
    },
    computed: {
        dataObj: function() {
            return this.initList(this.dataArray)
        }
    },
    watch: {
        activeIndex(val) {
            this.indexlist(this.dataObj[val]._uid);
        },
        index(index) {
            this.activeIndex = index;
            this.scrollContent(index);
        }
    },
    methods: {
        init() {
            this.scrollView = this.$refs.scrollView;

            this.contentOffsetTop = this.scrollView.getBoundingClientRect().top;

            if (this.index > 0) {
                this.indexlist(this.dataObj[this.index]._uid, false);
                this.scrollContent(this.index, false);
            }
        },

        // 首字母大写
        ucfirst(l1) {
            if (l1.length > 0) {
                var first = l1.substr(0, 1).toUpperCase();
                var spare = l1.substr(1, l1.length);
                return first + spare;
            }
        },

        // 在对象中搜索
        arraySearch(l1, l2) {
            // debugger;
            for (var name in PinYin) {
                if (PinYin[name].indexOf(l1) != -1) {
                    return this.ucfirst(name);
                    break;
                }
            }
            return '#';
        },

        // 汉字转拼音
        ConvertPinyin(l1) {
            var I1 = "";
            var reg = new RegExp('[a-zA-Z\- ]');
            for (var i = 0; i < 1; i++) {
                var val = l1.substr(i, 1);
                var name = this.arraySearch(val, PinYin);
                if (reg.test(val)) {
                    I1 += val;
                } else if (name !== false) {
                    I1 += name;
                }

            }
            I1 = I1.replace(/ /g, '-');
            while (I1.indexOf('--') > 0) {
                I1 = I1.replace('--', '-');
            }
            return I1;
        },
        initList(rows) {
            var indexs = [];
            for (var key in rows) {
                if (rows.hasOwnProperty(key)) {
                    var element = rows[key];
                    if (element.name) {
                        var _name = this.ConvertPinyin(trim(element.name));
                        var _M = _name.substr(0, 1)
                        var ret = indexs.filter(function(item) {
                            return item.key == _M
                        })
                        if (!ret.length) {
                            indexs.push({ key: _M, value: [element] });
                        } else {
                            ret[0].value.push(element);
                        }
                    }
                }
            }

            indexs = indexs.sort(function(a, b) {
                if (a.key == '#')
                    return 1;
                return a.key > b.key ? 1 : -1
            })

            return indexs;
        },

        indexlist(_uid, animate = true) {
            const navWidth = ~~this.$refs.nav.offsetWidth / 2;

            this.dataObj.every((item, index) => {
                if (item._uid === _uid) {
                    const navitem = this.$refs['navitem_' + index][0];
                    const scrollOffset = navitem.offsetLeft - navWidth + navitem.offsetWidth / 2;

                    this.scrollLeft(this.currentOffset, scrollOffset, animate, () => {
                        this.callback && this.callback(index);
                    });
                    return false;
                }
                return true;
            });
        },
        scrollContent(index, animate = true) {
            this.toggle = false;
            this.activeIndex = index;
            this.scrolling = true;

            const panel = this.$refs.scrollItem[index];
            const speed = animate && (window.navigator && window.navigator.userAgent || '').indexOf('MicroMessenger') < 0 ? 500 : 0;
            scrollTop(this.scrollView, this.currentPosition, panel.offsetTop, speed, () => {
                this.scrolling = false;
            });

            this.currentPosition = panel.offsetTop - this.$refs.navbox.offsetHeight;
        },
        scrollLeft(from, to, animate, callback) {
            const difference = Math.abs(from - to);
            const step = animate ? Math.ceil(difference / 600 * 50) : difference;
            const self = this;

            function scroll(start, end, step) {
                if (start === end) {
                    callback && callback();
                    return;
                }

                let d = (start + step > end) ? end : start + step;
                if (start > end) {
                    d = (start - step < end) ? end : start - step;
                }
                self.$refs.nav.scrollLeft = d;
                self.currentOffset = d;
                window.requestAnimationFrame(() => scroll(d, end, step));
            }

            scroll(from, to, step);
        }
    },
    mounted() {
        this.$nextTick(this.init);
    },
    destroyed() {
        this.scrollView.removeEventListener('scroll', this.scrollHandler);
        window.removeEventListener('resize', this.scrollHandler);
    }
}
</script>

<style lang="less">
.indexlist-title {
    height: 50px;
    line-height: 50px;
    font-size: 16px;
    color: #4242ff;
    text-align: left;
    background-color: #e4e4e4;
    padding-left: 7px;
}

@import "../../../styles/components/indexlist.less";
</style>
