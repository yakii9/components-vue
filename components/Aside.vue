<template>
    <div id="aside-container" class="aside" v-if="isActive" :class="[{ 'is-active': isActive }, placementClass]">
        <div class="modal-background" v-if="backdrop" @click="backdropClose"></div>
        <div ref="card" class="modal-card" :style="modalWidth">
            <header class="modal-card-head aside-header" v-if="showHeader">
                <p class="modal-card-title">{{ title }}</p>
                <i v-if="hasCloseButton" class="el-dialog__close el-icon el-icon-close close" @click="handleCancel" />
            </header>
            <section class="modal-card-body aside-body">
                <slot></slot>
            </section>
        </div>
    </div>
</template>
<script>
    import ModalMixin, { delayTime } from './ModalMixin';

    // 此组件使用hasCloseButton参数来控制组件是否具有关闭按钮
    // width参数来制定其Container的宽度
    // placement参数指定aside的打开方向
    // transition是面板打开时，动画效果的名称
    export default {
        mixins: [ModalMixin],

        props: {
            width: {
                type: Number,
                default: 450,
            },
            placement: {
                type: String,
                default: 'right',
            },
            transition: {
                type: String,
                default: 'el-fade-in-linear',
            },
            hasCloseButton: {
                type: Boolean,
                default: false
            }
        },

        computed: {
            placementClass() {
                if (this.placement && this.placement !== 'left') {
                    return `aside-${this.placement}`;
                }
                return null;
            },
            transitionName() {
                if (this.placement === 'right' && this.transition === 'fadeLeft') {
                    return 'fadeRight';
                }
                return this.transition;
            },
        },

        watch: {
            // 监视组件隐藏还是展示，从而添加动画效果
            isActive (newValue, oldValue) {
                const self = this
                if (newValue === true && oldValue === false) {
                    if (this.placement === 'left') {
                        setTimeout(() => {
                            $(self.$refs['card']).animate({
                                left: '10px'
                            }, delayTime)
                        }, 0)
                    }

                    setTimeout(() => {
                        $(self.$refs['card']).animate({
                            right: '10px',
                        }, delayTime)
                    }, 0)
                }
            }
        }
    };
</script>

<style lang="scss" scoped>
    @import "../controlview/Script_Script/variables.scss";

    .aside{
        position: fixed;
        z-index: 1986;
        bottom: 0;
        left: 0;
        right: 0;
        top: 0;
        transition: all .3s ease;
        .close{
            font-size: 21px;
            font-weight: 700;
            line-height: 1;
            color: #000;
            text-shadow: 0 1px 0 #fff;
            cursor: pointer;
            filter: alpha(opacity=20);
            opacity: .2;
            font-family: Arial,sans-serif;
        }
        .close:focus, .close:hover{
            filter: alpha(opacity=50);
            opacity: .5;
        }
        .modal-card{
            width: 450px;
            max-width: 100%;
            background-color: #ffffff;
            border-radius: 12px;
            margin: 0;
            position: absolute;
            top: 20px;
            bottom: 20px;
            box-shadow: 0px 0px 32px #888888;
            max-height: 100%;
            .modal-card-head, .modal-card-foot{
                border-radius: 0;
            }
        }
        &.aside-right{
            .modal-card{
                right: -2000px;
            }
        }

        &.aside-left{
            .modal-card{
                left: -2000px;
            }
        }
    }

    .modal{
        transition: all .3s ease;
        &.align-baseline{
            align-items: baseline;
        }
    }
    .modal-background{
        transition: opacity .3s ease;
    }
    .modal-card{
        box-shadow: 0 1px 4px rgba(0,0,0,.2);
        border-radius: 5px;
    }
    .modal-card-head, .modal-card-foot{
        background-color: #fff;
        padding: 16px;
        border-radius: 12px 12px 0 0 !important;
    }
    .modal-card-head{
        border-bottom: 1px solid rgba(120,130,140,.13);
    }
    .modal-card-foot{
        border-top: 1px solid rgba(120,130,140,.13);
        justify-content: flex-end;
    }
    .modal-card-title{
        font-size: 16px;
        font-weight: 500;
    }
    .align-baseline{
        .modal-card{
            margin-top: 120px;
        }
    }
    .modal-enter {
        opacity: 0;
    }

    .modal-leave-active {
        opacity: 0;
    }

    .modal-enter .modal-card,
    .modal-leave-active .modal-card {
        -webkit-transform: scale(1.1);
        transform: scale(1.1);
    }
    .close{
        cursor: pointer;
        filter: alpha(opacity=20);
        opacity: .2;
    }
    .close:focus, .close:hover{
        filter: alpha(opacity=50);
        opacity: .5;
    }

    .modal-confirm{
        .modal-card{
            width: 400px;
            .media{
                &.is-primary{
                    .media-left{
                        color: $primary-blue;
                    }
                }
                &.is-info{
                    .media-left{
                        color: $green;
                    }
                }
                &.is-warning{
                    .media-left{
                        color: $orange;
                    }
                }
                &.is-danger{
                    .media-left{
                        color: $red;
                    }
                }
            }
            .media-left{
                font-size: 20px;
            }
        }
    }

    .aside-header {
        height: 30px;
        background-color: $primary-blue;
        display: flex;
        justify-content: space-between;
        align-items: center;
        
        .modal-card-title {
            color: #ffffff;
        }
    }

    .aside-body {
        background-color: #ffffff;
    }
</style>

