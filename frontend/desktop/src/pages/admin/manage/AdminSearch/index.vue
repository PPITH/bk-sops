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
    <div class="admin-search" data-test-id="adminMessage_form_search">
        <div v-if="!showResultComp" class="search-wrapper">
            <p class="tips">{{ $t('输入项目名、模板ID或任务ID进行搜索') }}</p>
            <bk-input
                v-model.trim="searchStr"
                class="search-input"
                right-icon="bk-icon icon-search"
                @change="onChange"
                @enter="onSearchInput">
            </bk-input>
        </div>
        <search-result
            v-else
            :keyword="searchStr"
            :has-edit-perm="hasEditPerm"
            :edit-perm-loading="editPermLoading"
            @onSearch="onSearchInput">
        </search-result>
    </div>
</template>
<script>
    import SearchResult from './SearchResult.vue'

    export default {
        name: 'AdminSearch',
        components: {
            SearchResult
        },
        props: {
            hasEditPerm: Boolean,
            editPermLoading: Boolean
        },
        data () {
            return {
                showResultComp: false,
                searchStr: '',
                timer: null
            }
        },
        methods: {
            onChange () {
                if (this.timer) {
                    clearTimeout(this.timer)
                }
                this.timer = setTimeout(() => {
                    this.onSearchInput()
                }, 1000)
            },
            onSearchInput () {
                this.showResultComp = true
            }
        }
    }
</script>
<style lang="scss" scoped>
    .admin-search {
        position: relative;
        height: calc(100% - 60px);
    }
    .search-wrapper {
        position: absolute;
        top: 34%;
        left: 50%;
        transform: translateX(-50%);
        width: 588px;
        .tips {
            margin-bottom: 8px;
            font-size: 12px;
            color: #63656e;
        }
        ::v-deep .search-input {
            input{
                color: #313238;
                height: 48px;
                line-height: 48px;
            }
            .icon-search {
                font-size: 18px;
                color: #63656e;
            }
        }
    }
</style>
