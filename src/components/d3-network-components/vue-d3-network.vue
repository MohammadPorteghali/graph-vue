<script>
    import * as forceSimulation from 'd3-force'

    const d3 = Object.assign({}, forceSimulation);
    import svgRenderer from './components/svgRenderer.vue'
    import canvasRenderer from './components/canvasRenderer.vue'
    import saveImage from './lib/saveImage.js'
    import svgExport from './lib/svgExport.js'

    export default {
        name: 'Network',
        components: {
            canvasRenderer,
            svgRenderer
        },
        props: {
            net_nodes: {
                type: Array
            },
            net_links: {
                type: Array
            },
            options: {
                type: Object
            },
            nodeSym: {
                type: String
            },
            nodeCb: {
                type: Function
            },
            linkCb: {
                type: Function
            },
            customForces: {
                type: Object
            },
            selection: {
                type: Object,
                default: () => {
                    return {
                        nodes: {},
                        links: {}
                    }
                }
            },
        },
        data() {
            return {
                canvas: false,
                nodes: [],
                links: [],
                offset: {
                    x: 0,
                    y: 0
                },
                clientOffset: {
                    x: 0,
                    y: 0
                },
                force: 500,
                forces: {
                    Center: false,
                    X: 0.5,
                    Y: 0.5,
                    ManyBody: true,
                    Link: true
                },
                noNodes: false,
                strLinks: true,
                fontSize: 10,
                dragging: false,
                linkWidth: 1,
                nodeLabels: false,
                nodeWeight: false,
                linkLabels: false,
                starMode: false,
                physics: true,
                nodeSize: 5,
                mouseOfst: {
                    x: 0,
                    y: 0
                },
                size: {},
                padding: {
                    x: 0,
                    y: 0
                },
                simulation: null,
                nodeSvg: null,
                resizeListener: true
            }
        },
        render(createElement) {
            let ref = 'svg'
            let props = {}
            let renderer = 'svg-renderer'
            let bindProps = [
                'size',
                'nodes',
                'links',
                'selected',
                'linksSelected',
                'strLinks',
                'linkWidth',
                'nodeLabels',
                'nodeWeight',
                'linkLabels',
                'starMode',
                'physics',
                'fontSize',
                'labelOffset',
                'offset',
                'padding',
                'nodeSize',
                'noNodes'
            ]

            for (let prop of bindProps) {
                props[prop] = this[prop]
            }
            props.nodeSym = this.nodeSvg

            if (this.canvas) {
                renderer = 'canvas-renderer';
                ref = 'canvas';
                props.canvasStyles = this.options.canvasStyles;
            }

            return createElement('div', {
                attrs: {class: 'net'},
                on: {'mousemove': this.move, '&touchmove': this.move}
            }, [
                createElement(renderer, {
                    props, ref, on: {action: this.methodCall}
                })
            ])
        },
        created() {
            this.updateOptions(this.options)
            this.buildNodes(this.net_nodes)
            this.links = this.buildLinks(this.net_links);
            this.updateNodeSvg()
        },
        mounted() {
            this.options.size.w = this.$el.clientWidth;
            this.options.size.h = this.$el.clientHeight;
            if (this.options.strLinks) {
                this.strLinks = this.options.strLinks;
            }
            this.onResize();
            this.$nextTick(() => {
                this.animate()
            });
            if (this.resizeListener) window.addEventListener('resize', this.onResize)
        },
        beforeDestroy() {
            if (this.resizeListener) window.removeEventListener('resize', this.onResize)
        },
        computed: {
            selected() {
                return this.selection.nodes
            },
            linksSelected() {
                return this.selection.links
            },
            center() {
                return {
                    x: this.options.size.w / 2 + (this.options.size.w / 200) + this.offset.x,
                    y: this.options.size.h / 2 + (this.options.size.h / 200) + this.offset.y
                }
            },
            labelOffset() {
                return {
                    x: (this.nodeSize / 2) + (this.fontSize / 2),
                    y: this.fontSize / 2
                }
            }
        },
        watch: {
            net_nodes(newValue) {
                this.buildNodes(newValue);
                this.reset()
            },
            net_links(newValue, oldValue) {
                this.links = this.buildLinks(newValue);
                this.reset()
            },
            nodeSym() {
                this.updateNodeSvg()
            },
            options(newValue, oldValue) {
                this.updateOptions(newValue);
                if (oldValue.size && newValue.size) {
                    if ((oldValue.size.w !== newValue.size.w) || (oldValue.size.h !== newValue.size.h)) {
                        this.onResize();
                    }
                }
                this.animate()
            }
        },
        methods: {
            /**
             * update svg
             *
             * @return {void}
             */
            updateNodeSvg() {
                let svg = null;
                if (this.nodeSym) {
                    svg = svgExport.svgElFromString(this.nodeSym)
                }
                this.nodeSvg = svg
            },
            /**
             * call a method
             *
             * @return {void}
             */
            methodCall(action, args) {
                let method = this[action];
                if (method && typeof (method) === 'function') {
                    if (args) method(...args);
                    else method()
                } else {
                    console.error('Call to undefined method:', action);
                }
            },
            /**
             *  vue event wrapper
             *
             * @param action
             * @param args
             * @return {void}
             */
            emitEvent(action, args) {
                this.$emit('action', action, args)
            },
            onResize() {
                let size = this.options.size;
                if (!size || !size.w) this.options.size.w = this.$el.clientWidth;
                if (!size || !size.h) this.options.size.h = this.$el.clientHeight;
                this.padding.x = 0;
                this.padding.y = 0;
                // serach offsets of parents
                let vm = this;
                while (vm.$parent) {
                    this.padding.x += vm.$el.offsetLeft || 0;
                    this.padding.y += vm.$el.offsetTop || 0;
                    vm = vm.$parent
                }
                this.animate()
            },
            // -- Data
            updateOptions(options) {
                for (let op in options) {
                    if (this.hasOwnProperty(op)) {
                        this[op] = options[op]
                    }
                }
            },
            buildNodes(nodes) {
                let vm = this
                this.nodes = nodes.map((node, index) => {
                    // node formatter option
                    node = this.itemCb(this.nodeCb, node);
                    // index as default node id
                    if (!node.id) vm.$set(node, 'id', index);
                    // initialize node coords
                    if (!node.x) vm.$set(node, 'x', 0);
                    if (!node.y) vm.$set(node, 'y', 0);
                    // node default name
                    if (!node.name) vm.$set(node, 'name', 'node ' + node.id);
                    if (node.svgSym) {
                        node.svgIcon = svgExport.svgElFromString(node.svgSym);
                        if (!this.canvas && node.svgIcon && !node.svgObj) node.svgObj = svgExport.toObject(node.svgIcon)
                    }
                    return node
                })
            },

            buildLinks(links) {
                return links.concat().map((link) => {
                    // link formatter option
                    link = this.itemCb(this.linkCb, link);
                    // source and target for d3
                    link.source = link.sid;
                    link.target = link.tid;
                    return link
                })
            },
            itemCb(cb, item) {
                if (cb && typeof (cb) === 'function') item = cb(item)
                return item
            },
            // -- Animation
            simulate(nodes, links) {
                let forces = this.forces
                let sim = d3.forceSimulation()
                    .stop()
                    .alpha(0.5)
                    // .alphaMin(0.05)
                    .nodes(nodes)

                if (forces.Center !== false) sim.force('center', d3.forceCenter(this.center.x, this.center.y))
                if (forces.X !== false) {
                    sim.force('X', d3.forceX(this.center.x).strength(forces.X))
                }
                if (forces.Y !== false) {
                    sim.force('Y', d3.forceY(this.center.y).strength(forces.Y))
                }
                if (forces.ManyBody !== false) {
                    sim.force('charge', d3.forceManyBody().strength(-this.force))
                }
                if (forces.Link !== false) {
                    sim.force('link', d3.forceLink(links).id(function (d) {
                        return d.id
                    }))
                }
                sim = this.setCustomForces(sim)
                return sim
            },
            setCustomForces(sim) {
                let forces = this.customForces
                if (forces) {
                    for (let f in forces) {
                        let d3Func = this.getD3Func('force' + f)
                        if (d3Func) {
                            let args = forces[f]
                            sim.force('custom' + f, d3Func(...args))
                        }
                    }
                }
                return sim
            },
            getD3Func(name) {
                let func = d3[name]
                if (func && typeof (func) === 'function') return func
                return null
            },
            animate() {
                if (this.simulation) this.simulation.stop()
                if (this.forces.Link !== false) this.simulation = this.simulate(this.nodes, this.links)
                else this.simulation = this.simulate(this.nodes)
                this.simulation.restart()
            },
            reset() {
                this.animate();
                this.nodes = this.simulation.nodes();
                if (this.forces.links) this.links = this.simulation.force('link').links();
            },
            stop() {
                this.simulation.stop();
            },
            // -- Mouse Interaction
            move(event) {
                let pos = this.clientPos(event);
                if (this.dragging !== false) {
                    if (this.nodes[this.dragging]) {
                        if (this.physics === false) {
                            this.stop();
                        } else {
                            this.simulation.alpha(0.5);
                            this.simulation.restart();
                        }
                        this.nodes[this.dragging].fx = pos.x - this.mouseOfst.x;
                        this.nodes[this.dragging].fy = pos.y - this.mouseOfst.y;
                    }
                }
            },
            clientPos(event) {
                let x = (event.touches) ? event.touches[0].clientX : event.clientX
                let y = (event.touches) ? event.touches[0].clientY : event.clientY
                x = x || 0;
                y = y || 0;
                return {x, y}
            },
            dragStart(event, nodeKey) {
                this.dragging = (nodeKey === false) ? false : nodeKey;
                this.setMouseOffset(event, this.nodes[nodeKey]);
                if (this.dragging === false) {
                    if (this.physics === false) {
                        this.stop();
                    } else {
                        this.simulation.alpha(0.1);
                        this.simulation.restart();
                    }
                    this.setMouseOffset();
                }
            },
            dragEnd() {
                let node = this.nodes[this.dragging]
                if (node && !node.pinned) {
                    // unfix node position
                    node.fx = null
                    node.fy = null
                }
                this.dragStart(false)
            },
            // -- Render helpers
            nodeClick(event, node) {
                this.$emit('node-click', event, node)
            },
            linkClick(event, link) {
                this.$emit('link-click', event, link)
            },
            setMouseOffset(event, node) {
                let x = 0;
                let y = 0;
                if (event && node) {
                    let pos = this.clientPos(event);
                    x = (pos.x) ? pos.x - node.x : node.x;
                    y = (pos.y) ? pos.y - node.y : node.y;
                }
                this.mouseOfst = {x, y}
            },
            screenShot(name, bgColor, toSVG, svgAllCss) {
                let exportFunc
                let args = []
                if (this.canvas) {
                    toSVG = false
                    exportFunc = this.$refs.canvas.canvasScreenShot
                    args = [bgColor]
                } else {
                    exportFunc = this.$refs.svg.svgScreenShot
                    args = [toSVG, bgColor, svgAllCss]
                }
                if (toSVG) {
                    name = name || 'export.svg'
                }
                else {
                    name = name || 'export.png'
                }

                exportFunc((err, url) => {
                    if (!err) {
                        if (!toSVG) saveImage.save(url, name)
                        else saveImage.download(url, name)
                    }
                    this.$emit('screen-shot', err)
                }, ...args)
            }
        }
    }
</script>

<style>
.net {
    height: 100%;
    margin: 0;
}

.net-svg {
}

.node {
    stroke: rgba(18, 120, 98, 0.7);
    stroke-width: 3px;
    transition: fill 0.5s ease;
    fill: #dcfaf3;
}

.node.selected {
    stroke: #caa455;
}

.node.pinned {
    stroke: rgba(190, 56, 93, 0.6);
}

.link {
    stroke: rgba(18, 120, 98, 0.3);
}
.darkModeClass .link {
    stroke: rgba(255, 201, 65, 0.56);
}

.node,
.link {
    stroke-linecap: round;
}

.node:hover,
.link:hover {
    stroke: #be385d;
    stroke-width: 5px;
}
.darkModeClass .node:hover,.link:hover {
    stroke: #f9436d;
    stroke-width: 5px;
}

.link.selected {
    stroke: rgba(202, 164, 85, 0.6);
}
.darkModeClass .link.selected {
    stroke: rgba(240, 198, 94, 0.6);
}

.curve {
    fill: none;
}

.node-label {
    fill: #127862;
}

svg .link {
    marker-end: url(#mid_arrow);
}

.arrow:hover {
    fill: #ff4841;
}
.darkModeClass .arrow:hover {
    fill: #adb2ff;
}

.darkModeClass .nodeLabel {
    fill: #fff125;
}
.nodeLabel {
    fill: #1eaf79;
}

.fadeLink {
    stroke: rgba(152, 152, 152, 0.1);
}

.selectedLink {
    stroke: rgba(255, 251, 0, 0.8);
}

.fadeNode {
    stroke: rgba(152, 152, 152, 0.7);
    stroke-width: 3px;
    transition: fill 0.5s ease;
    fill: #a9a9a9;
}
.fadeNodeLabel {
    stroke-width: 3px;
    transition: fill 0.5s ease;
    fill: #9b9b9b;
}

.selectedNode {
    stroke: rgba(255, 227, 0, 0.7);
    stroke-width: 3px;
    transition: fill 0.5s ease;
    fill: #fffb00;
}

.selectedNodeLabel {
    transition: fill 0.5s ease;
    fill: #ff3700;
}

.darkModeClass .selectedNodeLabel {
    transition: fill 0.5s ease;
    fill: #62cfff;
}

.mainNode {
    stroke: rgba(7, 90, 246, 0.9);
    stroke-width: 3px;
    transition: fill 0.5s ease;
    fill: #30c5fa;
}

.hasPasswdNode {
    stroke: rgba(23, 83, 20, 0.9);
    stroke-width: 3px;
    transition: fill 0.5s ease;
    fill: #16fa00;
}

.hasBackupNode {
    stroke: rgba(20, 246, 0, 0.9);
    stroke-width: 3px;
    transition: fill 0.5s ease;
    fill: #16fa00;
}

.mainNodeLinks {
    stroke: rgba(36, 124, 240, 0.7);
}

.darkModeClass .mainNodeLinks {
    stroke: rgba(251, 84, 38, 0.76);
}

.node-weight-label {
    fill: #af2538;
}

.weight-labels {
    fill: #127862;
}

.dashLine {
    stroke-dasharray: 3, 5;
}

.contact {
    stroke-dasharray: 0 !important;
}
</style>

