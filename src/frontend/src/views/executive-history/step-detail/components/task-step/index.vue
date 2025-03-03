<!--
 * Tencent is pleased to support the open source community by making BK-JOB蓝鲸智云作业平台 available.
 *
 * Copyright (C) 2021 THL A29 Limited, a Tencent company.  All rights reserved.
 *
 * BK-JOB蓝鲸智云作业平台 is licensed under the MIT License.
 *
 * License for BK-JOB蓝鲸智云作业平台:
 *
 *
 * Terms of the MIT License:
 * ---------------------------------------------------
 * Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated
 * documentation files (the "Software"), to deal in the Software without restriction, including without limitation
 * the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and
 * to permit persons to whom the Software is furnished to do so, subject to the following conditions:
 *
 * The above copyright notice and this permission notice shall be included in all copies or substantial portions of
 * the Software.
 *
 * THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT
 * THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
 * AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF
 * CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS
 * IN THE SOFTWARE.
-->

<template>
    <div
        class="task-execute-bar-step"
        :class="{
            [data.displayStyle]: true,
            active: data.stepInstanceId === activeId,
        }"
        @mouseenter="handleShowPopover"
        @mouseleave="handleHidePopover"
        @click="handleSelect">
        <div class="step-wraper">
            <div class="step-icon">
                <Icon :type="data.icon" />
            </div>
            <img
                v-if="data.isDoing"
                class="loading-progress"
                src="/static/images/task-loading.png">
        </div>
        <Icon :type="data.lastStepIcon" svg class="step-next" />
        <component
            v-show="isShowPopover"
            ref="popover"
            :is="popoverCom"
            :data="data"
            :class="['task-status-bar-step-popover', arrowPlacement]"
            :style="popoverStyles"
            @on-update="handleTaskStatusUpdate"
            @on-close="handleClosePopover" />
    </div>
</template>
<script>
    import ApprovalView from './view/approval';
    import NormalView from './view/normal';
    import NotStartView from './view/no-start';

    let activeHandler = null;

    export default {
        props: {
            activeId: {
                type: [String, Number],
            },
            data: {
                type: Object,
                required: true,
            },
        },
        data () {
            return {
                isShowPopover: false,
                confirmReason: '',
                popoverPosition: {
                    top: 0,
                    left: 0,
                },
                arrowPlacement: 'top',
            };
        },
        computed: {
            popoverCom () {
                if (this.data.isApproval) {
                    return ApprovalView;
                }
                if (this.data.isNotStart) {
                    return NotStartView;
                }
                return NormalView;
            },
            popoverStyles () {
                const { top, left, zIndex } = this.popoverPosition;
                return {
                    position: 'absolute',
                    top: `${top}px`,
                    left: `${left + 72}px`,
                    'z-index': zIndex,
                };
            },
        },
        beforeDestroy () {
            this.destroyePopover();
        },
        methods: {
            handleSelect () {
                this.$emit('on-select', this.data);
            },
            handleTaskStatusUpdate (payload) {
                this.$emit('on-update', payload);
            },
            handleShowPopover () {
                if (activeHandler) {
                    activeHandler.handleClosePopover();
                }
                
                this.isShowPopover = true;
                activeHandler = this;
                let offset = 0;
                if (this.data.isApproval) {
                    offset = 15;
                }
                this.$nextTick(() => {
                    this.arrowPlacement = 'top';
                    const position = {
                        top: 0,
                        left: 0,
                        zIndex: window.__bk_zIndex_manager.nextZIndex(), // eslint-disable-line no-underscore-dangle
                    };
                    const $popoverTarget = this.$refs.popover.$el;
                    const windowHeight = window.innerHeight;
                    const { height } = $popoverTarget.getBoundingClientRect();
                    const { top, left } = this.$el.getBoundingClientRect();
                    position.left = left;
                    position.top = top + offset;
                    if (top + height + 20 > windowHeight) {
                        this.arrowPlacement = 'bottom';
                        position.top = top - height + 60;
                    }
                    if (position.top < 20) {
                        this.arrowPlacement = 'middle';
                        position.top = top - height / 2 + 30;
                    }
                    document.body.appendChild($popoverTarget);
                    this.popoverPosition = position;
                });
            },
            // 人工确认步骤——如果没有确认需要手动关闭
            // 其它步骤鼠标离开自动关闭
            handleHidePopover () {
                if (this.data.isApproval && this.data.displayStyle !== 'success') {
                    return;
                }
                this.isShowPopover = false;
            },
            handleClosePopover () {
                this.isShowPopover = false;
            },
            destroyePopover () {
                try {
                    activeHandler = null;
                    if (this.$refs.popover.$el && document.body.hasChildNodes(this.$refs.popover.$el)) {
                        document.body.removeChild(this.$refs.popover.$el);
                    }
                } catch {}
            },
        },
    };
</script>
<style lang='postcss'>
    .task-execute-bar-step {
        cursor: pointer;

        &.start,
        &.end {
            cursor: default;

            &:hover {
                background: #fff;
            }
        }

        &:hover {
            background: #f0f1f5;
        }

        &.active {
            background: #e1ecff;
        }

        &.loading {
            color: #63656e;

            .step-icon {
                background: #3a84ff;
            }

            .time {
                color: #3a84ff;
            }
        }

        &.disabled {
            cursor: default;
            background: transparent;

            .step-icon {
                background: #dcdee5;
            }
        }

        &.fail,
        &.forced,
        &.confirmForced {
            color: #979ba5;

            .step-icon {
                background: #ff5656;
            }
        }

        &.confirm {
            color: #979ba5;

            .step-icon {
                background: #ff9c01;
            }
        }

        .step-wraper {
            position: relative;
            z-index: 1;
            display: flex;
            justify-content: center;
            align-items: center;
            width: 60px;
            height: 60px;
        }

        .step-icon {
            display: flex;
            width: 34px;
            height: 34px;
            font-size: 14px;
            font-weight: 600;
            color: #fff;
            background: #2dcb9d;
            border-radius: 50%;
            align-items: center;
            justify-content: center;
        }

        .step-next {
            position: absolute;
            left: 50%;
            z-index: 0;
            font-size: 31px;
            color: #c4c6cc;
            transform: translate(-50%, -18px);
        }

        .loading-progress {
            position: absolute;
            top: 9px;
            left: 9px;
            display: block;
            width: 42px;
            height: 42px;
            animation: 'ani-rotate' 2s linear infinite;
        }
    }

    .task-status-bar-step-popover {
        /* position: relative; */
        font-size: 14px;
        line-height: 22px;
        color: #63656e;
        white-space: nowrap;
        background: #fff;
        border: 1px solid #dcdee5;
        border-radius: 2px;
        box-shadow: 0 0 5px 0 rgba(0, 0, 0, 0.09);

        /* &:after{
            content: '';
            position: absolute;
            top: 0;
            left: -15px;
            width: 20px;
            height: 60px;
        } */
    }
</style>
