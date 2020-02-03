<template>
    <div class="fullscreen network-container">
        <network v-if="!loading"
                 ref='net'
                 class="d3-network"
                 :net_nodes="network.nodes"
                 :net_links="network.links"
                 :options="network.options"
                 :selection="{nodes: network.selected, links: network.linksSelected}"
                 v-on:node-click="nodeClick"
                 v-on:link-click="linkClick"
                 v-on:screen-shot='screenShotDone'
                 v-on:action="NetworkEvent">
        </network>
    </div>
</template>

<script>
    import Network from './d3-network-components/vue-d3-network'
    import NetworkMenu from './d3-network-components/network-menu'
    import NetworkSelection from './d3-network-components/network-selection'
    import NetworkFilter from './d3-network-components/network-filter'

    const networkData = {
        "nodes": [
            {
                "id": 1,
                "name": "علی",
                "size": 5,
                "color": "#123456",
                "type": "full",
                "x": 0,
                "y": 0,
                "description": "hi i am ali"
            },
            {
                "id": 2,
                "name": "رضا",
                "size": 5,
                "color": "#2882d0",
                "type": "half",
                "x": 0,
                "y": 0,
                "description": "hi i am ali"
            },
            {
                "id": 3,
                "name": "محمد",
                "size": 5,
                "color": "#d0434e",
                "type": "half",
                "x": 0,
                "y": 0,
                "description": "hi i am ali"
            },
            {
                "id": 4,
                "name": "قاسم",
                "size": 2,
                "color": "#63914f",
                "type": "full",
                "x": 0,
                "y": 0,
                "description": "hi i am ali"
            },
            {
                "id": 5,
                "name": "محمد هاشم",
                "size": 8,
                "color": "#7c11cb",
                "type": "full",
                "x": 0,
                "y": 0,
                "description": "hi i am ali"
            },
            {
                "id": 6,
                "name": "علی",
                "size": 7,
                "color": "#e4e80b",
                "type": "half",
                "x": 0,
                "y": 0,
                "description": "hi i am ali"
            },
            {
                "id": 7,
                "name": "احمد رضا عابد زاده",
                "size": 4,
                "color": "#006869",
                "type": "full",
                "x": 0,
                "y": 0,
                "description": "hi i am ali"
            }
        ],
        "links": [
            {
                "id": 1,
                "sid": {
                    "id": 1,
                    "name": "علی",
                    "size": 5,
                    "color": "#123456",
                    "type": "full",
                    "x": 0,
                    "y": 0,
                    "description": "hi i am ali"
                },
                "target": {
                    "id": 2,
                    "name": "رضا",
                    "size": 5,
                    "color": "#2882d0",
                    "type": "half",
                    "x": 0,
                    "y": 0,
                    "description": "hi i am ali"
                },
                "size": 15,
                "color": "#f843f8",
                "label": "t1",
                "type": "full",
                "description": "t1 description"
            },
            {
                "id": 2,
                "sid": {
                    "id": 1,
                    "name": "علی",
                    "size": 5,
                    "color": "#123456",
                    "type": "full",
                    "x": 0,
                    "y": 0,
                    "description": "hi i am ali"
                },
                "target":
                    {
                        "id": 5,
                        "name": "محمد هاشم",
                        "size": 8,
                        "color": "#7c11cb",
                        "type": "full",
                        "x": 0,
                        "y": 0,
                        "description": "hi i am ali"
                    },
                "size": 5,
                "color": "#f80f00",
                "label": "t2",
                "type": "half",
                "description": "t2 description"
            },
            {
                "id": 3,
                "sid": {
                    "id": 1,
                    "name": "علی",
                    "size": 5,
                    "color": "#123456",
                    "type": "full",
                    "x": 0,
                    "y": 0,
                    "description": "hi i am ali"
                },
                "target": {
                    "id": 7,
                    "name": "احمد رضا عابد زاده",
                    "size": 4,
                    "color": "#006869",
                    "type": "full",
                    "x": 0,
                    "y": 0,
                    "description": "hi i am ali"
                },
                "size": 25,
                "color": "#00baf8",
                "label": "t3",
                "type": "full",
                "description": "t3 description"
            }
        ]
    };

    const networkDataId = {
        "nodes": [
            {
                "id": 1,
                "name": "علی",
                "size": 5,
                "color": "#123456",
                "type": "full",
                "x": 0,
                "y": 0,
                "description": "hi i am ali"
            },
            {
                "id": 2,
                "name": "رضا",
                "size": 5,
                "color": "#2882d0",
                "type": "half",
                "x": 0,
                "y": 0,
                "description": "hi i am ali"
            },
            {
                "id": 3,
                "name": "محمد",
                "size": 5,
                "color": "#d0434e",
                "type": "half",
                "x": 0,
                "y": 0,
                "description": "hi i am ali"
            },
            {
                "id": 4,
                "name": "قاسم",
                "size": 2,
                "color": "#63914f",
                "type": "full",
                "x": 0,
                "y": 0,
                "description": "hi i am ali"
            },
            {
                "id": 5,
                "name": "محمد هاشم",
                "size": 8,
                "color": "#7c11cb",
                "type": "full",
                "x": 0,
                "y": 0,
                "description": "hi i am ali"
            },
            {
                "id": 6,
                "name": "علی",
                "size": 7,
                "color": "#e4e80b",
                "type": "half",
                "x": 0,
                "y": 0,
                "description": "hi i am ali"
            },
            {
                "id": 7,
                "name": "احمد رضا عابد زاده",
                "size": 4,
                "color": "#006869",
                "type": "full",
                "x": 0,
                "y": 0,
                "description": "hi i am ali"
            }
        ],
        "links": [
            {
                "id": 1,
                "sid": 1,
                "tid": 2,
                "size": 15,
                "color": "#f843f8",
                "label": "t1",
                "type": "full",
                "description": "t1 description"
            },
            {
                "id": 2,
                "sid": 1,
                "tid":5,
                "size": 5,
                "color": "#f80f00",
                "label": "t2",
                "type": "half",
                "description": "t2 description"
            },
            {
                "id": 3,
                "sid": 1,
                "tid": 4,
                "size": 25,
                "color": "#00baf8",
                "label": "t3",
                "type": "full",
                "description": "t3 description"
            }
        ]
    };
    export default {
        name: 'Graph',
        components: {
            Network
        },
        data() {
            return {
                loading: true,
                network: {
                    common: false,
                    commonShow: false,
                    allNodes: [],
                    links: [],
                    selected: {},
                    linksSelected: {},
                    currentSelected: {
                        type: '',
                        data: {}
                    },
                    commonDetailsModal: {
                        visible: false,
                        searchKey: '',
                        detailsNodes: []
                    },
                    options: {
                        force: 3000,
                        nodeSize: 35,
                        strLinks: false,
                        nodeLabels: true,
                        nodeWeight: false,
                        linkLabels: false,
                        starMode: false,
                        physics: true,
                        canvas: false,
                        linkWidth: 2,
                        size: {
                            w: 1000,
                            h: 800
                        },
                        offset: {
                            x: 0,
                            y: 0
                        },
                    },
                    toolsModal: {
                        visible: false,
                        type: 'Config'
                    },
                    filterTableDoLayout: false,
                    selectModal: {
                        visible: false,
                    },
                    linkDetailModal: {
                        visible: false,
                        data: {}
                    },
                    svgChoice: {
                        visible: false,
                        data: {}
                    }
                }
            }
        },

        mounted() {
            this.$set(this.network.options.size, 'w', this.$el.clientWidth);
            this.$set(this.network.options.size, 'h', this.$el.clientHeight);
            this.getData();
        },
        methods: {
            /**
             * method to get mined data
             *
             * @return {void}
             */
            getData() {
                this.loading = true;
                let nodes = [];
                //compute nodes
                networkDataId.nodes.forEach((val, key) => {
                    nodes.push({
                        id: val.id,
                        name: val.name,
                        size: val.size,
                        color: val.color,
                        description: val.description,
                        type: val.type,
                        x: val.x,
                        y: val.y,
                        data: val
                    });
                });
                this.$set(this.network, 'nodes', nodes);
                this.$set(this.network, 'allNodes', nodes);
                this.$set(this.network, 'links', networkDataId.links);
                this.loading = false;
            }
            ,
            /**
             * fires when options change
             *
             * @param options
             * @return {void}
             */
            changeOptions(options) {
                this.$set(this.network, 'options', Object.assign({}, options));
            }
            ,
            /**
             * method to return selected node and selected edge
             *
             * @return {object}
             */
            selection() {
                return {
                    nodes: this.network.selected,
                    links: this.network.linksSelected
                }
            }
            ,
            /**
             * method for selection event
             *
             * @param {string} action
             * @param {array} args
             * @return {void}
             */
            selectionEvent(action, args) {
                this.methodCall(this, action, args);
                this.updateSelection();
            }
            ,
            /**
             * method for selection event
             *
             * @param {string} action
             * @param {array} args
             * @return {void}
             */
            NetworkEvent(action, args) {
                this.methodCall(this, action, args);
            }
            ,

            /**
             *  vue custom event handler
             *
             * @param vm
             * @param action
             * @param args
             * @return {void}
             */
            methodCall(vm, action, args) {
                let method = vm[action];
                if (typeof method === 'function') {
                    if (args) {
                        method(args);
                    } else {
                        method()
                    }
                } else {
                    console.error('Call to undefined method:', action);
                }
            }
            ,
            /**
             * method to update selection
             *
             * @return {void}
             */
            updateSelection() {

                this.showSelection = (Object.keys(this.network.selected).length | Object.keys(this.network.linksSelected).length);
            }
            ,
            /**
             * method for removing a link
             *
             * @param {object} link
             * @return {void}
             */
            removeLink(link) {
                this.unSelectLink(link.id);
                this.netLinks.splice(link.index, 1);
            }
            ,
            /**
             * method for rebuilding links
             *
             * @param {array} nodes
             * @return {object}
             */
            rebuildLinks(nodes) {
                if (!nodes) nodes = this.nodes;
                let links = this.rebuildLinksUtils(nodes, this.links);
                for (let link of links.removed) {
                    if (this.network.linksSelected[link.id]) {
                        delete (this.network.linksSelected[link.id])
                    }
                }
                return links.newLinks;
            }
            ,
            /**
             * method for removing node
             *
             * @param {number} nodeId
             * @return {void}
             */
            removeNode(nodeId) {
                this.removeNodeById(nodeId, this.nodes, (nodes) => {
                    if (nodes) {
                        this.netLinks = this.rebuildLinks(nodes);
                        this.deselectNode(nodeId);
                        this.nodes = this.rebuildNodes(this.netLinks, nodes);
                    }
                })
            }
            ,
            /**
             * method for fixing a a node
             *
             * @param {object} node
             * @return {void}
             */
            pinNode(node) {
                if (!node.pinned) {
                    node.pinned = true;
                    node.fx = node.x;
                    node.fy = node.y;
                } else {
                    node.pinned = false;
                    node.fx = null;
                    node.fy = null;
                }
                this.nodes[node.index] = node
            }
            ,
// -- Selection
            /**
             * select a node
             *
             * @param {object} node
             * @return {void}
             */
            selectNode(node) {
                this.clearSelection();
                this.network.selected[node.id] = node;
                this.network.currentSelected.data = node.data;
                this.network.currentSelected.type = 'node';
                this.network.selectModal.visible = true;
            }
            ,
            /**
             * select links of a node
             *
             * @return {void}
             */
            selectNodesLinks() {
                for (let link of this.network.links) {
                    // node is selected
                    if (this.network.selected[link.sid] || this.network.selected[link.tid]) {
                        this.selectLink(link)
                        // node is not selected
                    } else {
                        this.unSelectLink(link.id)
                    }
                }
                this.updateSelection()
            }
            ,
            /**
             * event for clicking a node
             *
             * @param {object} event
             * @param {object} node
             * @return {void}
             */
            nodeClick(event, node) {
                switch (this.tool) {
                    case 'pin':
                        this.pinNode(node);
                        break;
                    default: // selection tool
                        // is selected
                        if (this.network.selected[node.id]) {
                            this.deselectNode(node.id);
                            // is not selected
                        } else {
                            this.selectNode(node);
                        }
                        this.selectNodesLinks();
                        break;
                }
                this.updateSelection();
            }
            ,
            /**
             * event for clicking a link
             *
             * @param {object} event
             * @param {object} link
             * @return {void}
             */
            linkClick(event, link) {
                this.clearSelection();
                this.network.currentSelected.data = link;
                this.network.currentSelected.type = 'link';
                if (this.network.linksSelected[link.id]) {
                    this.unSelectLink(link.id);
                } else {
                    this.selectLink(link);
                }
                this.updateSelection()
            }
            ,

            /**
             * event for hovering a link
             *
             * @param {array} args
             * @return {void}
             */
            linkOver(args) {
                this.network.linkDetailModal.visible = true;
                this.network.linkDetailModal.data = args[1];
            }
            ,

            /**
             * event for Leaving a link
             *
             * @return {void}
             */
            linkLeave() {
                this.$set(this.network.linkDetailModal, 'visible', false);
            }
            ,

            /**
             * method for creating a parent node for a node
             *
             * @param {object} node
             * @return {void}
             */
            createParent(node) {
                let nodeId = this.lastNodeId + 1;
                let linkId = this.lastLinkId + 1;
                let nNode = this.newNode(nodeId);
                nNode.x = node.x + 50;
                nNode.y = node.y + 50;
                this.nodes = this.nodes.concat(nNode);
                this.lastNodeId++;
                this.netLinks = this.netLinks.concat(this.newLink(linkId, node.id, nodeId));
                this.lastLinkId++;
            }
            ,
            /**
             * method for selecting a link
             *
             * @param {object} link
             * @return {void}
             */
            selectLink(link) {
                this.$set(this.network.linksSelected, link.id, link);
                this.network.selectModal.visible = true;
            }
            ,
            /**
             * make an array unique
             *
             * @param {array} array
             * @param {string} type
             * @return {array}
             */
            arrayUnique(array, type = 'id') {
                let a = array.concat();
                for (let i = 0; i < a.length; ++i) {
                    for (let j = i + 1; j < a.length; ++j) {
                        if (type === 'id') {
                            if (a[i].id === a[j].id) {
                                a.splice(j--, 1);
                            }
                        } else if (type === 'tid-sid') {
                            if (a[i].tid === a[j].tid && a[i].sid === a[j].sid) {
                                a.splice(j--, 1);
                            }
                        } else {
                            if (a[i] === a[j]) {
                                a.splice(j--, 1);
                            }
                        }
                    }
                }

                return a;
            }
            ,
            /**
             * check if an item exist in a list
             *
             * @param {object} obj
             * @param {array} list
             * @return {boolean}
             */
            containsObject(obj, list) {
                let x;
                for (x in list) {
                    if (list.hasOwnProperty(x) && list[x] === obj) {
                        return true;
                    }
                }

                return false;
            }
            ,
            /**
             * deselect all selected objects
             *
             * @return {void}
             */
            clearSelection() {
                this.network.selected = {};
                this.network.linksSelected = {};
                this.network.selectModal.visible = false;
            }
            ,
            /**
             * deselect all extended nodes
             *
             * @return {void}
             */
            clearExtended() {
                this.network.extendedNodes = [];
            }
            ,
            /**
             * deselect a selected node by id
             *
             * @param {number} nodeId
             * @return {void}
             */
            deselectNode(nodeId) {
                if (this.network.selected[nodeId]) {
                    delete (this.network.selected[nodeId]);
                }
                this.selectNodesLinks();
            }
            ,
            /**
             * deselect a selected link by id
             *
             * @param {number} linkId
             * @return {void}
             */
            unSelectLink(linkId) {
                if (this.network.linksSelected[linkId]) {
                    delete (this.network.linksSelected[linkId]);
                }
            }
            ,
            /**
             * method to show menu
             *
             * @param {boolean} show
             * @return {void}
             */
            setShowMenu(show) {
                this.showMenu = show;
                this.showHint = false;
            }
            ,
            /**
             * show mails of a node with it's filtered time
             *
             * @param {object} node
             * @return {void}
             */
            nodeMails(node) {
                this.$router.selectedNode = {
                    data: node,
                    dateTimeRange: this.$router.selectData.dateTimeRange
                };
//            this.$router.push({name: 'account', params: {accountId: node.id}});
            }
            ,

            //utilities

            /**
             *  find links by node => [ links ] | null
             *
             *  @param {number} nodeId
             *  @param {array} links
             *  @return {boolean}
             */
            findLinks(nodeId, links) {
                let nodeLinks = [];
                for (let link of links) {
                    if (link.sid === nodeId || link.tid === nodeId) nodeLinks.push(link)
                }
                return (nodeLinks.length) ? nodeLinks : null
            }
            ,

            /**
             *  find node by id=> [ links ] | null
             *
             *  @param {number} nodeId
             *  @param {array} nodes
             *  @return {object}
             */
            findNode(nodes, nodeId) {
                let index = this.nodeExists(nodeId);
                if (index) {
                    return nodes[index]
                }
                return null
            }
            ,

            /**
             * removes node by id => () => ( [newNodes] )
             *
             * @param nodeId
             * @param nodes
             * @param cb
             */
            removeNodeById(nodeId, nodes, cb) {
                let index = nodes.findIndex(
                    (node) => {
                        return node.id === nodeId
                    }
                );
                if (index > -1) {
                    nodes.splice(index, 1);
                    cb(nodes)
                } else {
                    cb(null)
                }
            }
            ,

            /**
             *  removes orphaned links => { newLinks, removed }
             *
             * @param nodes
             * @param links
             * @return {{newLinks: Array, removed: Array}}
             */
            rebuildLinksUtils(nodes, links) {
                let newLinks = [];
                let removed = [];
                for (let link of links) {
                    if (this.nodeExists(link.sid, nodes) && this.nodeExists(link.tid, nodes)) {
                        newLinks.push(link);
                    } else {
                        removed.push(link);
                    }
                }
                return {newLinks, removed}
            }
            ,

            /**
             *  removes unlinked nodes => [ newNodes ]
             *
             * @param links
             * @param nodes
             * @return {Array}
             */
            rebuildNodes(links, nodes) {
                let newNodes = [];
                for (let node of nodes) {
                    if (this.isLinked(node.id, links)) {
                        newNodes.push(node)
                    }
                }
                return newNodes
            }
            ,

            /**
             *  finds node by id => boolean
             *
             * @param nodeId
             * @param nodes
             * @return {boolean}
             */
            nodeExists(nodeId, nodes) {
                let index = nodes.findIndex(
                    (node) => {
                        return node.id === nodeId
                    }
                );
                return (index > -1)
            }
            ,

            /**
             *  Checks if node is linked => boolean
             *
             * @param nodeId
             * @param links
             * @return {boolean}
             */
            isLinked(nodeId, links) {
                let index = links.findIndex(
                    (link) => {
                        return (link.tid === nodeId || link.sid === nodeId);
                    }
                );
                return (index > -1);
            }
            ,

            /**
             *  link formatter
             *
             * @param id
             * @param sid
             * @param tid
             * @return {{id: *, sid: *, tid: *}}
             */
            newLink(id, sid, tid) {
                return {id, sid, tid}
            }
            ,

            /**
             *  generates random links => [ links ]
             *
             * @param nodes
             * @param maxLinks
             * @return {Array}
             */
            makeRandomLinks(nodes, maxLinks) {
                let links = [];
                let id = 0;
                for (let node of nodes) {
                    let total = Math.floor(Math.random() * maxLinks);
                    for (let i = 0; i <= total; i++) {
                        let target = Math.floor(Math.random() * nodes.length);
                        let source = node.id;
                        id++;
                        links.push(this.newLink(id, source, target))
                    }
                }
                return links
            }
            ,

            /**
             *  random node name
             *
             * @return {string}
             */
            newNodeName() {
                return Math.random().toString(36).substring(7)
            }
            ,

            /**
             *  node formatter
             *
             * @param nodeId
             * @return {{id: *, name}}
             */
            newNode(nodeId) {
                return {id: nodeId, name: this.newNodeName()}
            }
            ,

            /**
             *  generates random nodes => [ nodes ]
             *
             * @param maxNodes
             * @return {object}
             */
            makeRandomNodes(maxNodes) {
                return Array.apply(null, {length: maxNodes})
                    .map((value, index) => {
                        return this.newNode(index)
                    })
            }
            ,

            /**
             *  vue event wrapper
             *
             * @param vm
             * @param action
             * @param args
             * @return {Component|*}
             */
            emitEvent(vm, action, args) {
                if (vm.$data.conf && vm.$data.conf.allEventsAs) {
                    let evName = vm.$data.conf.allEventsAs;
                    return vm.$emit(evName, action, args);
                }
                return vm.$emit(action, ...args);
            }
            ,

            /**
             * show screen shot modal
             *
             * @return {void}
             */
            screenShot() {
                this.network.svgChoice.visible = !this.network.svgChoice.visible;
            }
            ,

            /**
             * take screen shot from graph
             *
             * @return {void}
             */
            takeScreenShot() {
                this.network.svgChoice.visible = false;
                this.$notify.info({
                    title: 'Exporting',
                    message: 'SVG is Exporting',
                    duration: 2000
                });
                this.$refs.net.screenShot(null, null, this.network.svgChoice.data)
            }
            ,

            /**
             * fires when screen shot is done
             *
             * @param {string} err
             * @return {void}
             */
            screenShotDone(err) {
                if (err) {
                    this.$notify.error({
                        title: 'Error',
                        message: err,
                        duration: 5000
                    });
                } else {
                    this.$notify({
                        title: 'Success',
                        message: 'Export Completed',
                        type: 'success',
                        duration: 3000
                    });
                }
            }
            ,
            /**
             * show tool modal
             *
             * @return {void}
             */
            showToolModal(val) {
                this.network.toolsModal.type = val;
                if (val === 'Common') {
                    this.$set(this.network.commonDetailsModal, 'visible', true);
                    this.$nextTick().then(() => {
                        this.commonModalSearch();
                    });
                } else {
                    this.network.toolsModal.visible = true;
                    if (val === 'Filter') {
                        setTimeout(() => {
                            this.$nextTick().then(() => {
                                this.network.filterTableDoLayout = !this.network.filterTableDoLayout;
                            });
                        })
                    }
                }
            },
        }
    }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
    .fullscreen {
        height: 100vh;
    }

    .network-container {
        background-color: #e8e8e8;
    }

    .d3-network {
        height: calc(100vh - 10px);
    }
</style>
