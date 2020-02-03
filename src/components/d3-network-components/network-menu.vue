<template>
    <div id="menu">
        <ul class="net-menu">
            <!--<li>-->
            <!--<el-checkbox v-model.number="opts.force" v-on:input="change" :min="1" :max="100000" :step="1"/>-->
            <!--<label>Physics</label>-->
            <!--</li>-->
            <li>
                <label>Force:<span>{{ opts.force }}</span></label>
                <el-slider v-model.number="opts.force" v-on:input="change" :min="1" :max="100000" :step="1"/>
            </li>
            <li>
                <label>Size Width:<span>{{ opts.size.w }}</span></label>
                <el-slider v-model.number="opts.size.w" v-on:input="change" :min="100" :max="10000" :step="2"/>
            </li>
            <li>
                <label>Size Height:<span>{{ opts.size.h }}</span></label>
                <el-slider v-model.number="opts.size.h" v-on:input="change" :min="100" :max="10000" :step="2"/>
            </li>
            <li>
                <label>
                    Offset X:
                    <span>
                        {{ opts.offset.x }}
                    </span>
                </label>
                <el-slider v-model.number="opts.offset.x" v-on:input="change" :min="-10000" :max="10000"
                           :step="2"/>
            </li>
            <li>
                <label>Offset Y:<span>{{ opts.offset.y }}</span></label>
                <el-slider v-model.number="opts.offset.y" v-on:input="change" :min="-10000" :max="10000"
                           :step="2"/>
            </li>
            <li>
                <label>Node Size:<span>{{ opts.nodeSize }}</span></label>
                <el-slider v-model.number="opts.nodeSize" v-on:input="change" :min="3" :max="500" :step="1"/>
            </li>
            <li>
                <label>Link Thickness:<span>{{ opts.linkWidth }}</span></label>
                <el-slider v-model.number="opts.linkWidth" v-on:input="change" :min="1" :max="40" :step="1"/>
            </li>
            <li>
                <el-checkbox v-model="opts.nodeLabels" v-on:change="change">
                    Show Node Names
                </el-checkbox>
            </li>
            <li>
                <el-checkbox v-model="opts.nodeWeight" v-on:change="change">
                    Show Node Weight
                </el-checkbox>
            </li>
            <!--<li>-->
            <!--<el-checkbox v-model="opts.linkLabels" v-on:change="change"/>-->
            <!--<label>Show Link Values</label>-->
            <!--</li>-->
            <li v-if="opts.nodeLabels || opts.nodeWeight || opts.linkLabels">
                <label>Font Size:<span>{{ opts.fontSize }}</span></label>
                <el-slider v-model="opts.fontSize" v-on:input="change" :min="10" :max="50" :step="1"/>
            </li>
            <li>
                <el-checkbox v-model="opts.strLinks" v-on:change="change">
                    Straight Links
                </el-checkbox>
            </li>
            <li>
                <el-checkbox v-model="opts.starMode" v-on:change="change">
                    Star Mode
                </el-checkbox>
            </li>
        </ul>
    </div>
</template>
<script>
export default {
    name: 'NetworkMenu',
    props: ['links', 'nodes', 'options'],
    data() {
        let data = Object.assign({}, {
            nodes: [],
            links: [],
            showMenu: false,
            selected: {},
            showSelection: false,
            linksSelected: {},
            options: {
                canvas: false,
                size: {
                    w: 500,
                    h: 500
                },
                force: 350,
                offset: {
                    x: 0,
                    y: 0
                },
                nodeSize: 20,
                linkWidth: 1,
                nodeLabels: false,
                linkLabels: false,
                nodeWeight: false,
                starMode: false,
                physics: true,
                strLinks: true
            }
        });
        return {
            opts: data.options,
            showNodeValues: false,
            showLinkValues: false,
            setts: null
        }
    },
    created() {
        this.opts = this.options;
        this.setts = this.settings;
    },
    methods: {
        /**
         * emit changes to parent component
         *
         * @return {void}
         */
        change() {
            this.$emit('options', this.opts)
        },
        /**
         * reset configs
         *
         * @return {void}
         */
        reset() {
            this.opts = Object.assign({}, {
                canvas: false,
                size: {
                    w: 500,
                    h: 500
                },
                force: 350,
                offset: {
                    x: 0,
                    y: 0
                },
                nodeSize: 20,
                linkWidth: 1,
                nodeLabels: false,
                linkLabels: false,
                strLinks: true
            });
            this.options.width = this.$el.clientWidth
            this.options.height = this.$el.clientHeight
            this.$emit('reset', this.options)
        }
    }
}
</script>
<style src="./assets/css/icons.css"></style>
<style scoped>
.debug {
    font-size: 0.5em;
    list-style: none;
}

.net-menu {
    padding: 1em;
    list-style: none;
}

.net-menu li {
    margin: 0.5em 0;
}

.net-menu li label {
    font-size: 0.85em;
    display: block;
}

.net-menu li label span {
    font-weight: normal;
}

.net-menu li input + label {
    display: inline;
    margin-left: 0.5em;
}

ul.net-menu + ul.net-menu {
    border-left: none;
}
</style>

