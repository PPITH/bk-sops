/**
* Tencent is pleased to support the open source community by making 蓝鲸智云PaaS平台社区版 (BlueKing PaaS Community
* Edition) available.
* Copyright (C) 2017 THL A29 Limited, a Tencent company. All rights reserved.
* Licensed under the MIT License (the "License"); you may not use this file except in compliance with the License.
* You may obtain a copy of the License at
* http://opensource.org/licenses/MIT
* Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on
* an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the
* specific language governing permissions and limitations under the License.
*/
<template>
    <page-header class="operation-header">
        <div class="head-left-area">
            <i class="bk-icon icon-arrows-left back-icon" @click="onBack"></i>
            <div class="operation-title">{{$t('任务执行')}}</div>
            <div class="bread-crumbs-wrapper">
                <span
                    class="path-item name-ellipsis"
                    v-for="(path, index) in nodeNav"
                    :key="path.id"
                    v-bk-overflow-tips
                    :title="showNodeList.includes(index) ? path.name : ''">
                    <span v-if="!!index && showNodeList.includes(index) || index === 1">/</span>
                    <span v-if="showNodeList.includes(index)" class="node-name" :title="path.name" @click="onSelectSubflow(path.id)">
                        {{path.name}}
                    </span>
                    <span class="node-ellipsis" v-else-if="index === 1">...</span>
                </span>
            </div>
            <router-link
                v-if="isShowViewProcess"
                class="common-icon-jump-link"
                v-bk-tooltips="{
                    content: $t('task_查看流程'),
                    placements: ['top']
                }"
                target="_blank"
                :to="getTplURL()">
            </router-link>
            <span v-if="stateStr" :class="['task-state', state]">{{ stateStr }}</span>
        </div>
        <div class="operation-container" slot="expand">
            <div class="task-operation-btns" v-show="isTaskOperationBtnsShow">
                <div
                    v-for="operation in taskOperationBtns"
                    :key="operation.action"
                    v-bk-tooltips="{
                        content: operation.text,
                        placements: ['top']
                    }"
                    :class="[
                        'operation-btn',
                        { 'btn-permission-disable': !hasPermission(['task_operate'], instanceActions) }
                    ]">
                    <bk-button
                        :class="[
                            operation.action === 'revoke' ? 'revoke-btn' : 'execute-btn'
                        ]"
                        theme="default"
                        hide-text="true"
                        :icon="'common-icon ' + operation.icon"
                        :key="operation.action"
                        :loading="operation.loading"
                        :disabled="operation.disabled"
                        v-cursor="{ active: !hasPermission(['task_operate'], instanceActions) }"
                        :data-test-id="`taskExcute_form_${operation.action}Btn`"
                        @click="onOperationClick(operation.action)">
                    </bk-button>
                </div>
            </div>
            <div class="task-params-btns">
                <i
                    :class="[
                        'params-btn',
                        'common-icon-enter-config',
                        {
                            actived: nodeInfoType === 'modifyParams'
                        }
                    ]"
                    v-bk-tooltips="{
                        content: $t('任务入参'),
                        placements: ['top'],
                        hideOnClick: false
                    }"
                    @click="onTaskParamsClick('modifyParams', $t('任务入参'))">
                </i>
                <i
                    :class="[
                        'params-btn',
                        'solid-eye',
                        'common-icon-solid-eye',
                        {
                            actived: nodeInfoType === 'viewNodeDetails'
                        }
                    ]"
                    v-bk-tooltips="{
                        content: $t('查看节点详情'),
                        placements: ['top']
                    }"
                    @click="onTaskParamsClick('viewNodeDetails', $t('节点详情'))">
                </i>
                <bk-popover placement="bottom-left" theme="light" ext-cls="operate-tip">
                    <i class="bk-icon icon-more drop-icon-ellipsis"></i>
                    <template slot="content">
                        <p class="operate-item" @click="onTaskParamsClick('operateFlow', $t('操作记录'))">
                            {{ $t('操作记录') }}
                        </p>
                        <p
                            v-if="state !== 'CREATED'"
                            class="operate-item"
                            @click="onTaskParamsClick('globalVariable', $t('全局变量'))">
                            {{ $t('全局变量') }}
                        </p>
                        <p class="operate-item" @click="onTaskParamsClick('templateData', 'Code')">
                            {{ 'Code' }}
                        </p>
                        <p v-if="adminView && engineVer === 1" class="operate-item" @click="onTaskParamsClick('taskExecuteInfo')">
                            {{ $t('流程信息') }}
                        </p>
                        <p v-if="adminView && engineVer === 2" class="operate-item" @click="$emit('onInjectGlobalVariable')">
                            {{ $t('注入全局变量') }}
                        </p>
                    </template>
                </bk-popover>
            </div>
        </div>
    </page-header>
</template>
<script>
    import permission from '@/mixins/permission.js'
    import PageHeader from '@/components/layout/PageHeader.vue'
    import { mapState } from 'vuex'

    export default {
        name: 'TaskOperationHeader',
        components: {
            PageHeader
        },
        mixins: [permission],
        props: [
            'nodeInfoType',
            'templateSource',
            'project_id',
            'template_id',
            'primitiveTplId',
            'primitiveTplSource',
            'nodeNav',
            'instanceActions',
            'taskOperationBtns',
            'adminView',
            'engineVer',
            'state',
            'stateStr',
            'isBreadcrumbShow',
            'isTaskOperationBtnsShow',
            'isShowViewProcess',
            'paramsCanBeModify'
        ],
        data () {
            return {
                showNodeList: [0, 1, 2]
            }
        },
        computed: {
            ...mapState({
                hideHeader: state => state.hideHeader,
                view_mode: state => state.view_mode
            })
        },
        watch: {
            nodeNav (val) {
                if (val.length > 3) {
                    this.showNodeList = [0, val.length - 1, val.length - 2]
                } else {
                    this.showNodeList = [0, 1, 2]
                }
            }
        },
        methods: {
            getTplURL () {
                let routerData = ''
                const templateId = this.primitiveTplId || this.template_id
                // business 兼容老数据
                if (this.primitiveTplSource === 'business' || this.primitiveTplSource === 'project') {
                    routerData = `/template/view/${this.project_id}/?template_id=${templateId}`
                } else if (this.primitiveTplSource === 'common') {
                    routerData = `/template/common/view/${this.project_id}/?template_id=${templateId}&common=1`
                }
                return routerData
            },
            onSelectSubflow (id) {
                this.$emit('onSelectSubflow', id)
            },
            onOperationClick (action) {
                this.$emit('onOperationClick', action)
            },
            onTaskParamsClick (type, name) {
                this.$emit('onTaskParamsClick', type, name)
            },
            onBack () {
                // 如果被嵌入了，则像父页面发送事件
                if (this.hideHeader) {
                    window.parent.postMessage({ eventName: 'goBackEvent' }, '*')
                    return
                }
                if (this.view_mode === 'appmaker') {
                    return this.$router.push({
                        name: 'appmakerTaskHome',
                        params: { type: 'edit', app_id: this.$route.params.app_id, project_id: this.project_id },
                        query: { template_id: this.$route.query.template_id }
                    })
                }
                if (this.$route.path.indexOf('/function') === 0) {
                    return this.$router.push({
                        name: 'functionHome'
                    })
                }
                if (this.$route.path.indexOf('/audit') === 0) {
                    return this.$router.push({
                        name: 'auditHome'
                    })
                }
                // 当任务执行页由创建任务路由过来时，应该返回到任务列表页
                const isFromCreate = this.$route.query.from === 'create'
                if (!isFromCreate && this.$route.name === 'taskExecute' && window.history.length > 2) {
                    return this.$router.back()
                }
                this.$router.push({
                    name: 'taskList',
                    params: { project_id: this.project_id }
                })
            }
        }
    }
</script>
<style lang="scss" scoped>
@import '@/scss/config.scss';

.operation-header {
    display: flex;
    justify-content: space-between;
    padding: 0 20px 0 10px;
    .head-left-area {
        display: flex;
        align-items: center;
        .back-icon {
            font-size: 28px;
            color: #3a84ff;
            cursor: pointer;
        }
    }
    .operation-title {
        font-size: 14px;
        color: #313238;
    }
    .bread-crumbs-wrapper {
        margin-left: 10px;
        font-size: 0;
        .path-item {
            display: inline-block;
            font-size: 14px;
            overflow: hidden;
            &.name-ellipsis {
                max-width: 700px;
                overflow: hidden;
                white-space: nowrap;
                text-overflow: ellipsis;
            }
            .node-name {
                margin: 0 4px;
                font-size: 14px;
                color: #3a84ff;
                cursor: pointer;
            }
            .node-ellipsis {
                margin-right: 4px;
            }
            &:first-child {
                .node-name {
                    margin-left: 0px;
                }
            }
            &:last-child {
                .node-name {
                    &:last-child {
                        color: #313238;
                        cursor: text;
                    }
                }
            }
        }
    }
    .common-icon-jump-link {
        color: #3a84ff;
        margin: 0 8px 0 4px;
    }
    .task-state {
        display: inline-block;
        margin-left: 10px;
        padding: 0 6px;
        height: 20px;
        line-height: 20px;
        font-size: 12px;
        color: #63656e;
        border-radius: 10px;
        background-color: #dcdee5;
        &.EXPIRED,
        &.CREATED {
            color: #63656e;
        }
        &.FINISHED {
            background-color: #cceed9;
            color: #2dcb56;
        }
        &.RUNNING,
        &.READY {
            background-color: #cfdffb;
            color: #3a84ff;
        }
        &.SUSPENDED, &.NODE_SUSPENDED {
            background-color: #ffe8c3;
            color: #d78300;
        }
        &.FAILED {
            background-color: #f2d0d3;
            color: #ea3636;
        }
        &.REVOKED {
            background-color: #f2d0d3;
            color: #ea3636;
        }
    }
    .operation-container {
        display: flex;
        align-items: center;
        height: 100%;
        .task-operation-btns,
        .task-params-btns {
            float: left;
            .bk-button {
                border: none;
                background: transparent;
                cursor: pointer;
            }
            ::v-deep .bk-icon {
                float: initial;
                top: 0;
                & + span {
                    margin-left: 0;
                }
            }
            .common-icon-branchs {
                font-size: 16px;
            }
        }
        .task-operation-btns {
            display: flex;
            margin-right: 35px;
            line-height: initial;
            border-right: 1px solid #dde4eb;
            .operation-btn {
                margin-right: 35px;
                height: 32px;
                line-height: 32px;
                font-size: 14px;
                &.btn-permission-disable {
                    background: transparent !important;
                }
            }
            .execute-btn {
                width: 140px;
                color: #ffffff;
                background: #3a84ff; // 覆盖 bk-button important 规则
                &:hover {
                    background: #699df4; // 覆盖 bk-button important 规则
                }
                &.is-disabled {
                    color: #ffffff; // 覆盖 bk-button important 规则
                    opacity: 0.4;
                    cursor: no-drop;
                }
                &.btn-permission-disable {
                    border: 1px solid #e6e6e6;
                }
                ::v-deep .bk-button-loading div {
                    background: #ffffff;
                }
            }
            .revoke-btn {
                padding: 0;
                background: transparent; // 覆盖 bk-button important 规则
                color: #ea3636;
                &:hover {
                    color: #c32929;
                }
                &.is-disabled {
                    color: #d8d8d8;
                }
                ::v-deep .common-icon-stop {
                    font-size: 24px;
                    margin-top: 10px;
                }
            }
        }
        .task-params-btns {
            .params-btn {
                margin-right: 25px;
                padding: 0;
                color: #979ba5;
                font-size: 14px;
                cursor: pointer;
                &.actived {
                    color: #63656e;
                }
                &:hover {
                    color: #63656e;
                }
            }
            .common-icon-enter-config {
                font-size: 18px;
            }
            .back-button {
                background: #ffffff;
                border: 1px solid #c4c6cc;
            }
            .drop-icon-ellipsis {
                font-size: 18px;
                font-weight: 600;
                color: #979ba5;
                cursor: pointer;
                &:hover {
                    color: #63656e;
                }
            }
        }
    }
    ::v-deep .bk-button .bk-icon {
        font-size: 14px;
    }
}
</style>
<style lang="scss">
.operate-tip {
    .tippy-tooltip {
        padding: 4px 0;
    }
    .operate-item {
        width: 160px;
        height: 40px;
        display: block;
        line-height: 40px;
        padding-left: 10px;
        color: #313238;
        font-size: 12px;
        cursor: pointer;
        &:hover {
            color: #3a84ff;
            background: #eaf3ff;
        }
    }
}
</style>
