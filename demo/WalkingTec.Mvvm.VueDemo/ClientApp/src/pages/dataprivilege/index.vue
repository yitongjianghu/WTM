<template>
    <card>
        <wtm-search-box :ref="searchRefName" :events="searchEvent" :formOptions="SEARCH_DATA" />
        <!-- 操作按钮 -->
        <wtm-but-box :assembly="assembly" :action-list="actionList" :selected-data="selectData" :events="actionEvent" />
        <!-- 列表 -->
        <wtm-table-box :attrs="{...searchAttrs, actionList}" :events="{...searchEvent, ...actionEvent}" />
        <!-- 弹出框 -->
        <dialog-form :is-show.sync="dialogIsShow" :dialog-data="dialogData" :status="dialogStatus" @onSearch="onHoldSearch" />
        <!-- 导入 -->
        <upload-box :is-show.sync="uploadIsShow" @onImport="onImport" @onDownload="onDownload" />
    </card>
</template>

<script lang='ts'>
import { Component, Vue } from "vue-property-decorator";
import { Action, State } from "vuex-class";
import searchMixin from "@/vue-custom/mixin/search";
import actionMixin from "@/vue-custom/mixin/action-mixin";
import store from "./store/index";
import DialogForm from "./views/dialog-form.vue";
// 查询参数/列表 ★★★★★
import { ASSEMBLIES, TABLE_HEADER } from "./config";

@Component({
    name: "dataprivilege",
    mixins: [searchMixin(TABLE_HEADER), actionMixin(ASSEMBLIES)],
    store,
    components: { DialogForm }
})
export default class Index extends Vue {
    @Action
    deleted;
    @Action
    getPrivileges;
    @State
    getPrivilegesData;

    get SEARCH_DATA() {
        return {
            formProps: {
                "label-width": "75px",
                inline: true
            },
            formItem: {
                TableName: {
                    type: "select",
                    label: "权限名称",
                    props: {
                        clearable: true
                    },
                    children: this.getPrivilegesData
                },
                DpType: {
                    type: "radioGroup",
                    label: "权限类型",
                    span: 8,
                    children: [
                        {
                            Value: 0,
                            Text: "用户组权限"
                        },
                        {
                            Value: 1,
                            Text: "用户权限"
                        }
                    ]
                }
            }
        };
    }

    created() {
        this.getPrivileges();
    }

    /**
     * 单个删除
     * @param params
     */
    onDelete(params) {
        this.onConfirm().then(() => {
            const parameters = {
                ModelName: params.TableName,
                Id: params.TargetId,
                Type: params.DpType
            };
            this.deleted(parameters).then(res => {
                this["$notify"]({
                    title: "删除成功",
                    type: "success"
                });
                this["onHoldSearch"]();
            });
        });
    }
}
</script>
