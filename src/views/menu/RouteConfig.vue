<template>
    <section id="route_config">
        <el-tree
            class="filter-tree"
            :data="menu"
            :props="{children: 'children',label: labelHandler}"
            :highlight-current="true"
            default-expand-all
            :expand-on-click-node="false"
            :filter-node-method="filterNode"
            :render-content="renderContent">
        </el-tree>

        <el-dialog title="路由配置" :visible.sync="handlerVis" width="30%" align="left">
            <el-form :model="handlerForm" class="dialog-form" size="mini">
                <template v-for="item in classInfo.properties">
                    <el-form-item v-if="item.isObject === '0'" :key="item.id" :label="item.des" :prop="item.name" :inline="true" label-width="105px">
                        <el-input v-model="handlerForm[item.name]" v-if="item.isObject === '0'" type="text" size="small"></el-input>
                    </el-form-item>
                    <template v-else>
                        <el-form-item :label="item.des" :key="item.id" label-width="100px">
                        </el-form-item>
                        <el-form-item v-for="subItem in item.obj.properties" :key="subItem.id" :label="subItem.des" :prop="subItem.name" :inline="true" label-width="105px">
                            <el-input v-model="handlerForm[item.name][subItem.name]" type="text" size="small"></el-input>
                        </el-form-item>
                    </template>
                </template>
                <el-form-item :inline="true" label-width="105px">
                    <el-button size="mini" type="primary" @click="submit">提交</el-button>
                    <el-button size="mini" type="danger"  @click="cancel">取消</el-button>
                </el-form-item>
            </el-form>
        </el-dialog>

    </section>
</template>

<script>
import DynamicRoutes from '@/utils/dynamic-routes';
export default {
    name: "RouteConfig",
    data() {
        return {
            menu: [],
            handlerVis: false, // 操作框显示控制标志
            handlerForm: {
                meta: {}
            }, //操作表单
            classInfo: {}, //对应实体类信息
        }
    },
    created() {
        this.menu = this.$store.state.menuList;
        this.classInfo = this.$store.state.classInfo;
    },
    methods: {
        /**
         * 过滤节点
         * @param value
         * @param data
         * @returns {boolean}
         */
        filterNode(value, data) {
            if (!value) return true;
            return data.label.indexOf(value) !== -1;
        },
        /**
         * 处理渲染的树label值
         * @param data
         * @returns {*}
         */
        labelHandler(data) {
            return data.meta.title;
        },
        /**
         * 树节点的内容区的渲染 Function
         * @param h
         * @param node
         * @param data
         * @param store
         * @returns {*}
         */
        renderContent(h, {node, data, store}) {
            return (
                <span class="custom-tree-node">
                    <span>{node.label}</span>
                    <span>
                        <el-button size="mini" type="text" on-click={() => this.append(data)}>
                            <i class="el-icon-circle-plus-outline"/>
                        </el-button>
                        <el-button size="mini" type="text" on-click={() => this.edit(data)}>
                            <i class="el-icon-edit"/>
                        </el-button>
                        <el-button size="mini" type="text" on-click={() => this.remove(node, data)}>
                            <i class="el-icon-remove-outline"/>
                        </el-button>
                    </span>
                </span>
            );
        },
        /**
         * 添加
         * @param data
         */
        append(data) {
            this.handlerVis = true;
            this.handlerForm.parentId = data.id;
        },
        /**
         * 编辑
         * @param data
         */
        edit(data) {
            this.handlerVis = true;
            this.handlerForm = JSON.parse(JSON.stringify(data));
        },
        /**
         * 提交新增或者更新
         */
        submit() {
            let type = "POST";
            let path = `/identity/sysRoutes/`;
            if (this.handlerForm.id) {
                type = "PUT";
                path += `${this.handlerForm.id}id`;
            }
            this.$http(type, path, Object.assign({},this.handlerForm)).then(() => this.getLastestMenu());
        },
        cancel() {
            this.handlerVis = false;
        },
        getLastestMenu() {
            this.$http('POST', `/identity/sysRoutes/list`).then(data => {
                sessionStorage.setItem("menu",JSON.stringify(data));
                this.$store.commit("getMenu",data);
                DynamicRoutes.transfer(data);
                this.$router.addRoutes(data);
                this.menu = this.$store.state.menuList;
                this.handlerVis = false;
            })
        },
        remove(node, data) {
            this.$confirm('确认删除该菜单么？', '确认信息', {
                    distinguishCancelAndClose: true,
                    confirmButtonText: '确认',
                    cancelButtonText: '取消'
                }).then(() => {
                    this.$http('DELETE', `/identity/sysRoutes/${data.id}id`).then(() => this.getLastestMenu());
                }).catch(action => this.$message({type: 'success', message: '取消成功'}));
        }
    }
}
</script>

<style>
.filter-tree {
    font-size: 14px;
}
#route_config {
    padding: 20px 0 0 20px;
    width: 40%;
}
.custom-tree-node {
    flex: 1;
    display: flex;
    align-items: center;
    justify-content: space-between;
    font-size: 14px;
    padding-right: 8px;
}
.dialog-form {
    padding-right: 50px;
}

.btn-right {
    float: right;
}
</style>
