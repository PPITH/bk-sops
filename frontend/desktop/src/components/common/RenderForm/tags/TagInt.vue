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
    <div class="tag-int">
        <div v-if="formMode">
            <el-input-number
                :min="min"
                :max="max"
                :controls="false"
                :disabled="!editable || disabled"
                :placeholder="placeholder"
                v-model="intValue"
                @blur="handleBlur">
            </el-input-number>
            <span v-show="!validateInfo.valid" class="common-error-tip error-info">{{validateInfo.message}}</span>
        </div>
        <span v-else class="rf-view-value">{{(value === 'undefined' || value === '') ? '--' : value}}</span>
    </div>
</template>
<script>
    import '@/utils/i18n.js'
    import i18n from '@/config/i18n/index.js'
    import { getFormMixins } from '../formMixins.js'

    export const attrs = {
        placeholder: {
            type: String,
            required: false,
            default: '',
            desc: 'placeholder'
        },
        min: {
            type: Number,
            required: false,
            default: -Infinity,
            desc: i18n.t('最大值')
        },
        max: {
            type: Number,
            required: false,
            default: Infinity,
            desc: i18n.t('最小值')
        },
        disabled: {
            type: Boolean,
            required: false,
            default: false,
            desc: i18n.t('禁用组件')
        },
        value: {
            type: [Number, String],
            required: false,
            default: 0
        }
    }
    export default {
        name: 'TagInt',
        mixins: [getFormMixins(attrs)],
        computed: {
            intValue: {
                get () {
                    return Number(this.value)
                },
                set (val) {
                    val = val || 0
                    this.updateForm(val)
                }
            }
        },
        methods: {
            handleBlur () {
                this.emit_event(this.tagCode, 'blur', this.value)
                this.$emit('blur', this.value)
            }
        }
    }
</script>
<style lang="scss" scoped>
.tag-int {
    .el-input-number {
        width: 100%;
    }
    ::v-deep .el-input__inner {
        height: 32px;
        line-height: 32px;
        text-align: left;
    }
}
</style>
