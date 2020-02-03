<template>
    <div id="selection" class="notification"  style="color: #1f1f1f">
        <el-button icon="el-icon-close" class="close" v-on:click='emit("clearSelection")'>
        </el-button>
        <h4>Selected {{selected.type}} :
        </h4>
        <el-row justify="start" type="flex">
            <el-tag type="success" v-if="selected.type==='node'" size="small">{{selected.data.public_name}}</el-tag>
            <div v-else>From
                <el-button type="success" v-on:click="nodeClick(selected.data.source)" size="small">
                    {{selected.data.source.data.public_name}}
                </el-button>
                To
                <el-button type="info" v-on:click="nodeClick(selected.data.target)" size="small">
                    {{selected.data.target.data.public_name}}
                </el-button>
            </div>
        </el-row>
        <div class="mini">
            <div v-if="selected.type=='node'">
                <el-row justify="center" type="flex">
                    <el-col>
                        Public Name:
                    </el-col>
                    <el-col>
                        <el-tag type="info" size="small">
                            {{selected.data.public_name}}
                        </el-tag>
                    </el-col>
                </el-row>
                <el-row justify="center" type="flex" v-if="selected.data.category">
                    <el-col>
                        Category:
                    </el-col>
                    <el-col>
                        <el-tag type="info" size="small">
                            {{selected.data.category.name}}
                        </el-tag>
                    </el-col>
                </el-row>
                <el-row justify="center" type="flex">
                    <el-col>
                        Email:
                    </el-col>
                    <el-col>
                        <el-tag type="info" size="small">
                            {{selected.data.email}}
                        </el-tag>
                    </el-col>
                </el-row>
                <el-row justify="start" type="flex">
                    <el-col :span="8">
                        Has Password:
                    </el-col>
                    <el-col :span="4">
                        <div v-if="selected.data.imported">
                            <el-tag type="success" size="small">
                                True
                            </el-tag>
                        </div>
                        <el-tag v-else type="danger" size="small">
                            False
                        </el-tag>
                    </el-col>
                </el-row>
                <el-row justify="start" type="flex">
                    <el-col :span="8">
                        Has Backup:
                    </el-col>
                    <el-col :span="4">
                        <div v-if="selected.data.has_backup">
                            <el-tag type="success" size="small">
                                True
                            </el-tag>
                        </div>
                        <el-tag v-else type="danger" size="small">
                            False
                        </el-tag>
                    </el-col>
                    <el-col :span="14">
                        <el-tooltip :enterable="false"content="Draw New Graph" placement="top">
                            <el-button icon="el-icon-share" v-on:click="drawNew(selected)" type="success" size="mini">
                            </el-button>
                        </el-tooltip>
                        <el-tooltip :enterable="false"content="Extend Graph" placement="top">
                            <el-button icon="el-icon-plus" v-on:click="extendNode(selected)" type="primary" size="mini">
                            </el-button>
                        </el-tooltip>
                        <!--<el-tooltip :enterable="false"content="Emails" placement="top">-->
                            <!--<el-button icon="el-icon-message" v-on:click="nodeMails(selected)" type="warning" size="mini">-->
                            <!--</el-button>-->
                        <!--</el-tooltip>-->
                    </el-col>
                </el-row>
                <el-row justify="center" type="flex">
                    <el-col :span="8">
                        receive :
                        <br/>
                        <el-tag type="info" size="small">
                            {{selected.data.weight.receive}}
                        </el-tag>
                    </el-col>
                    <el-col :span="8">
                        send :
                        <br/>
                        <el-tag type="info" size="small">
                            {{selected.data.weight.send}}
                        </el-tag>
                    </el-col>
                    <el-col :span="8">
                        total :
                        <br/>
                        <el-tag type="info" size="small">
                            {{selected.data.weight.send + selected.data.weight.receive}}
                        </el-tag>
                    </el-col>
                </el-row>
            </div>
            <div v-if="selected.type==='link'">
                <el-row justify="center" type="flex" v-if="selected.data.total > 0">
                    <el-col :span="12">
                        Total:
                    </el-col>
                    <el-col :span="12">
                        <el-tag type="info" size="small">
                            {{selected.data.total}}
                        </el-tag>
                    </el-col>
                </el-row>
                <el-row justify="center" type="flex" v-if="selected.data.BCC > 0">
                    <el-col>
                        BCC:
                    </el-col>
                    <el-col>
                        <el-tag type="info" size="small">
                            {{selected.data.BCC}}
                        </el-tag>
                    </el-col>
                </el-row>
                <el-row justify="center" type="flex" v-if="selected.data.CC > 0">
                    <el-col>
                        CC:
                    </el-col>
                    <el-col>
                        <el-tag type="info" size="small">
                            {{selected.data.CC}}
                        </el-tag>
                    </el-col>
                </el-row>
                <el-row justify="center" type="flex" v-if="selected.data.TO > 0">
                    <el-col>
                        To:
                    </el-col>
                    <el-col>
                        <el-tag type="info" size="small">{{selected.data.TO}}
                        </el-tag>
                    </el-col>
                </el-row>
            </div>
        </div>
    </div>
</template>
<script>
export default {
    name: 'NetworkSelection',
    props: ['data', 'selected'],
    methods: {
        /**
         * emit an action to parent component
         *
         * @param {string} action
         * @param {string|object|array} args
         * @return {void}
         */
        emit(action, args) {
            this.$emit('action', action, args)
        },
        /**
         * fire when node clicked
         *
         * @param {object} node
         * @return {void}
         */
        nodeClick(node) {
            this.emit('selectNode', node);
            this.emit('selectNodesLinks', []);
            this.emit('updateSelection', []);
        },
        /**
         * fire when click on extend node
         *
         * @param {object} node
         * @return {void}
         */
        extendNode(node) {
            this.emit('extendNode', node.data);
            this.emit("clearSelection", []);
        },
        /**
         * fire when click on draw new graph
         *
         * @param {object} node
         * @return {void}
         */
        drawNew(node) {
            vuex.dispatch('setMinerProperties', {accounts: [node.data.id]});
            this.emit("clearExtended", []);
            this.emit("clearSelection", []);
            this.emit('getData', '');
        },
        /**
         * fire when click on node mails
         *
         * @param {object} node
         * @return {void}
         */
        nodeMails(node) {
            this.emit('nodeMails', node.data);
        }
    },
    computed: {
        links() {
            return this.data.links
        },
        nodes() {
            return this.data.nodes
        }
    }
}
</script>
<style scoped>
button.icon {
    background-color: #fff;
    border-style: none;
    font-size: 1em;
    height: 1.5em;
    width: 1.5em;
    line-height: 1.25em;
    border-radius: 50%;
    padding: 0;
    margin: 0;
    color: #1aad8d;
}

button.big {
    margin-bottom: 1em;
}

button.big span {
    font-size: 2em;
}

.mini {
    font-size: 0.8em;
}

.title {
    color: #1aad8d;
    font-weight: bold;
}

table {
    border-collapse: collapse;
}

tr {
    border-bottom: solid 1px #d3d3d3;
}

th {
    border-bottom: solid 2px #d3d3d3;
}

th,
td {
    padding: 0 0.5em;
    text-align: left;
}

.mini-list,
ul.list {
    display: inline-block;
    list-style: none;
}

.notification {
    position: absolute;
    right: 3em;
    z-index: 100;
    width: auto;
    padding: 1em 3em 1em 2em;
    border-radius: 0.5em;
    box-shadow: 1px 1px 2px rgba(0, 0, 0, 0.7);
    border: #1aad8d solid 2px;
    background-color: #fff;
    box-shadow: 2px 2px 4px rgba(0, 0, 0, 0.8);
}

.notification .mini-list {
    max-height: 10em;
    overflow-y: scroll;
    overflow-x: hidden;
}
</style>
