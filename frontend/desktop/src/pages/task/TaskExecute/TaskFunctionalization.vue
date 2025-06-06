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
    <div class="functionalization-wrapper">
        <task-create-header
            :steps="stepList"
            :common="common"
            :title="$t('职能化认领')"
            :instance-name="instanceName"
            :current-step="3"
            :is-functional="true"
            :project_id="project_id"
            :template_id="templateId">
        </task-create-header>
        <div class="form-area">
            <div class="task-info">
                <span class="task-info-title">{{ $t('任务信息') }}</span>
                <div class="task-info-division-line"></div>
                <div class="common-form-item">
                    <label class="required">{{ $t('任务名称') }}</label>
                    <div class="common-form-content">
                        <bk-input
                            class="task-name"
                            name="taskName"
                            v-model="name"
                            v-validate="taskNameRule"
                            :maxlength="stringLength.TASK_NAME_MAX_LENGTH"
                            :show-word-limit="true">
                        </bk-input>
                        <span class="common-error-tip error-msg">{{ veeErrors.first('taskName') }}</span>
                    </div>
                </div>
            </div>
            <div class="param-info">
                <div class="param-info-title">
                    <span>
                        {{ $t('参数信息') }}
                    </span>
                </div>
                <div class="param-info-division-line"></div>
                <div v-if="!isVariableEmpty" class="form-wrapper" v-bkloading="{ isLoading: isConfigLoading, opacity: 1, zIndex: 100 }">
                    <TaskParamEdit
                        ref="TaskParamEdit"
                        :constants="pipelineData.constants"
                        @onChangeConfigLoading="changeLoading">
                    </TaskParamEdit>
                </div>
                <NoData v-else></NoData>
            </div>
        </div>
        <div class="action-wrapper">
            <bk-button
                class="preview-button"
                @click="onShowPreviewDialog">
                {{ $t('预览') }}
            </bk-button>
            <bk-button
                theme="primary"
                :class="['task-claim-button', {
                    'btn-permission-disable': !hasPermission(['task_claim'], instanceActions)
                }]"
                :loading="isSubmit"
                :disabled="isConfigLoading"
                v-cursor="{ active: !hasPermission(['task_claim'], instanceActions) }"
                @click="onTaskClaim">
                {{ $t('认领') }}
            </bk-button>
        </div>
        <bk-dialog
            :value="previewDialogShow"
            :mask-close="false"
            :auto-close="false"
            :header-position="'left'"
            :has-footer="false"
            :ext-cls="'common-dialog'"
            :title="$t('任务流程预览')"
            width="1000"
            @cancel="onClose">
            <NodePreview
                v-if="canvasShow"
                ref="nodePreviewRef"
                :preview-data-loading="previewDataLoading"
                :canvas-data="canvasData"
                :preview-bread="previewBread"
                @onNodeClick="onNodeClick"
                @onSelectSubflow="onSelectSubflow">
            </NodePreview>
            <div slot="footer">
                <bk-button theme="primary" @click="onClose">{{ $t('关闭') }}</bk-button>
            </div>
        </bk-dialog>
    </div>
</template>
<script>
    import { mapState, mapActions } from 'vuex'
    import tools from '@/utils/tools.js'
    import i18n from '@/config/i18n/index.js'
    import { NAME_REG, STRING_LENGTH } from '@/constants/index.js'
    import permission from '@/mixins/permission.js'
    import TaskCreateHeader from '../TaskCreateHeader.vue'
    import NoData from '@/components/common/base/NoData.vue'
    import TaskParamEdit from '../TaskParamEdit.vue'
    import NodePreview from '../NodePreview.vue'
    import { formatCanvasData } from '@/utils/checkDataType'

    const STEP_DICT = [
        {
            step: 'selectnode',
            icon: 1,
            title: i18n.t('选择节点')
        },
        {
            step: 'paramfill',
            icon: 2,
            title: i18n.t('填写参数')
        },
        {
            step: 'taskexecute',
            icon: 3,
            title: i18n.t('职能化认领')
        },
        {
            step: 'taskexecute',
            icon: 4,
            title: i18n.t('去执行')
        }
    ]

    export default {
        name: 'TaskFunctionalization',
        inject: ['reload'],
        components: {
            NoData,
            TaskCreateHeader,
            TaskParamEdit,
            NodePreview
        },
        mixins: [permission],
        props: [
            'common',
            'project_id',
            'templateId',
            'instanceId',
            'instanceFlow',
            'instanceName',
            'instanceActions'
        ],
        data () {
            return {
                stepList: STEP_DICT.slice(0),
                isSubmit: false,
                isConfigLoading: false,
                previewDialogShow: false,
                canvasShow: false,
                previewDataLoading: false,
                name: this.instanceName,
                nodeSwitching: false,
                pipelineData: JSON.parse(this.instanceFlow),
                previewData: JSON.parse(this.instanceFlow),
                stringLength: STRING_LENGTH,
                taskNameRule: {
                    required: true,
                    max: STRING_LENGTH.TASK_NAME_MAX_LENGTH,
                    regex: NAME_REG
                },
                previewBread: []
            }
        },
        computed: {
            ...mapState('project', {
                'projectId': state => state.project_id,
                'projectName': state => state.projectName
            }),
            isVariableEmpty () {
                return Object.keys(this.pipelineData.constants).length === 0
            },
            canvasData () {
                return formatCanvasData('preview', this.previewData)
            }
        },
        watch: {
            /** HACK
             * magicbox V2.1.8 版本，dialog 组件在切换为显示状态后，画布组件开始渲染，
             * 弹窗内容区域 display 属性仍为 none，在 $nextTick 里才变更，
             * 导致画布组件首次渲染时因为容器高度为 0，连线不能正确渲染位置
             */
            previewDialogShow (val) {
                if (val) {
                    setTimeout(() => {
                        this.canvasShow = true
                    }, 0)
                } else {
                    this.canvasShow = false
                }
            }
        },
        methods: {
            ...mapActions('task/', [
                'claimFuncTask'
            ]),
            updateCanvas () {
                this.previewDataLoading = true
                this.$nextTick(() => {
                    this.previewDataLoading = false
                })
            },
            changeLoading (val) {
                this.isConfigLoading = val
            },
            onTaskClaim () {
                if (this.isSubmit) return

                if (!this.hasPermission(['task_claim'], this.instanceActions)) {
                    const resourceData = {
                        task: [{
                            id: this.instanceId,
                            name: this.instanceName
                        }],
                        project: [{
                            id: this.projectId,
                            name: this.projectName
                        }]
                    }
                    this.applyForPermission(['task_claim'], this.instanceActions, resourceData)
                    return
                }

                this.isSubmit = true
                this.$validator.validateAll().then(async (result) => {
                    if (!result) return
                    const formData = {}
                    if (this.$refs.TaskParamEdit) {
                        const variables = await this.$refs.TaskParamEdit.getVariableData()
                        for (const key in variables) {
                            const value = variables[key]
                            if (value.source_type !== 'component_outputs' && value.show_type === 'show') {
                                formData[key] = value.value
                            }
                        }
                    }
                    const data = {
                        name: this.name,
                        instance_id: this.instanceId,
                        project_id: this.project_id,
                        constants: formData
                    }
                    try {
                        const res = await this.claimFuncTask(data)
                        if (res.result) {
                            this.reload()
                        }
                    } catch (e) {
                        console.log(e)
                    } finally {
                        this.isSubmit = false
                    }
                })
            },
            onShowPreviewDialog () {
                this.previewBread.push({
                    name: this.name,
                    data: this.previewData
                })
                this.previewDialogShow = true
            },
            onClose () {
                this.previewDialogShow = false
                this.previewDataLoading = false
                this.previewData = tools.deepClone(this.pipelineData)
                this.previewBread = []
            },
            onSelectSubflow (data, index) {
                this.previewData = data
                this.previewBread.splice(index + 1)
                this.updateCanvas()
            },
            onNodeClick (id) {
                const activity = this.previewData.activities[id]
                if (!activity || activity.type !== 'SubProcess') {
                    return
                }

                const previewData = activity.pipeline
                this.previewBread.push({
                    data: previewData,
                    name: activity.name
                })
                this.previewData = previewData
                this.updateCanvas()
            }
        }
    }
</script>
<style lang="scss" scoped>
@import '@/scss/config.scss';
@import "@/scss/mixins/scrollbar.scss";

.functionalization-wrapper {
    position: relative;
    height: calc(100vh - 100px);
    background-color: #ffffff;
}
.form-area {
    padding-top: 20px;
    height: calc(100% - 72px);
    overflow: auto;
    @include scrollbar;
}
.task-info, .param-info {
    margin: 0 40px 20px 40px;
    .task-info-title, .param-info-title {
        font-size: 14px;
        font-weight: 600;
        color: #313238;
    }
    .task-info-division-line, .param-info-division-line {
        margin: 5px 0 30px;
        height: 1px;
        border: 0px;
        background-color: #cacedb;
    }
    .common-form-item {
        label {
            color: #313238;
            font-weight: normal;
        }
    }
}
.param-info  {
    padding-bottom: 80px;
}
.task-param-wrapper {
    width: 100%;
}
.form-wrapper {
    min-height: 200px;
}
.action-wrapper {
    height: 72px;
    line-height: 72px;
    border-top: 1px solid #cacedb;
    background-color: #ffffff;
    text-align: left;
    box-shadow: 0 -3px 4px 0 rgba(64,112,203,0.06);
    .preview-button {
        padding: 0px;
        margin-left: 40px;
        width: 90px;
        height: 32px;
        line-height: 32px;
        color: #313238;
    }
    .task-claim-button {
        width: 140px;
    }
}
.task-name {
    max-width: 500px;
}
::v-deep .bk-dialog-body {
    height: 420px;
    background-color: #f4f7fa;
}
::v-deep .pipeline-canvas{
    .tool-wrapper {
        top: 20px;
        left: 40px;
    }
}
::v-deep .render-form {
    .el-input__inner,
    .el-tree,
    .el-date-editor,
    .el-textarea__inner,
    .el-input-number,
    .tag-input,
    .el-date-editor,
    .el-cascader,
    .el-select,
    .user-selector-layout,
    ::v-deep .ip-search-wrap {
        max-width: 598px;
    }
    ::v-deep .module-form  {
        .bk-form-content,
        .bk-input-number,
        .bk-select {
            max-width: 598px;
        }
    }
    .el-radio__label,
    .checkbox-item {
        max-width: 160px;
        margin-right: 24px;
    }
}

</style>
