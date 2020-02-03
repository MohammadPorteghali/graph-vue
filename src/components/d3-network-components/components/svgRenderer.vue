<template>
    <svg ref="svg"
         id="network-graph"
         :width="size.w"
         :height="size.h"
         class="net-svg"
         v-on:mouseup='emit("dragEnd",[$event])'
         v-on:touchend.passive='emit("dragEnd",[$event])'
         v-on:touchstart.passive=''>
        <defs>
            <marker id="mid_arrow" markerwidth="20" markerheight="20" orient="auto" viewBox="-10 -10 20 20" refX="12">
                <path class="arrow" fill="#ff4841" d="M-10,-10 L10,0 L-10,10 z"></path>
            </marker>
        </defs>
        <g class="nodes" id="l-nodes" v-if="!noNodes">
            <template v-for="(node,key) in nodes">
                <svg v-if='svgIcon(node)'
                     :key='key'
                     :viewBox='svgIcon(node).attrs.viewBox'
                     :width='getNodeSize(node, "width")'
                     :height='getNodeSize(node, "height")'
                     v-on:click='emit("nodeClick",[$event,node])'
                     v-on:touchend.passive='emit("nodeClick",[$event,node])'
                     v-on:mousedown.prevent='$event.stopImmediatePropagation();emit("dragStart",[$event,key])'
                     v-on:touchstart.passive='emit("dragStart",[$event,key])'
                     v-on:mouseup='emit("dragEnd",[$event])'
                     :x='node.x - getNodeSize(node, "width") / 2'
                     :y='node.y - getNodeSize(node, "height") / 2'
                     :style='nodeStyle(node)'
                     :title="node.name"
                     :class='"node-svg " + nodeClass(node)'
                     v-html='svgIcon(node).data'>
                </svg>
                <circle v-else
                        :key='key'
                        :r="getNodeSize(node) / 8"
                        v-on:click='emit("nodeClick",[$event,node])'
                        v-on:touchend.passive='emit("nodeClick",[$event,node])'
                        v-on:mousedown.prevent='mouseDown($event,key)'
                        v-on:touchstart.passive='emit("dragStart",[$event,key])'
                        v-on:mouseup='emit("dragEnd",[$event])'
                        :cx="node.x"
                        :cy="node.y"
                        :style='nodeStyle(node)'
                        :title="node.name"
                        :class="nodeClass(node)">
                </circle>
            </template>
        </g>
        <g class="links" id="l-links" v-if="strLinks">
            <line v-for="link, index in links"
                  :x1='link.source.x'
                  :y1='link.source.y'
                  :x2='link.target.x'
                  :y2='link.target.y'
                  :stroke-width='linkWidth'
                  v-on:click='emit("linkClick",[$event,link])'
                  v-on:touchstart.passive='emit("linkClick",[$event,link])'
                  v-on:mouseover='emit("emitEvent",["linkOver",[$event,link]])'
                  v-on:mouseleave='emit("emitEvent",["linkLeave",[$event,link]])'
                  :class='linkClass(link)'>
            </line>
        </g>
        <g class="links" id="l-links" v-else="v-else">
            <path v-for="link in links" ouch
                  :d="curve(link)"
                  v-on:click='emit("linkClick",[$event,link])'
                  v-on:touchstart.passive='emit("linkClick",[$event,link])'
                  v-on:mouseover='emit("emitEvent",["linkOver",[$event,link]])'
                  v-on:mouseleave='emit("emitEvent",["linkLeave"])'
                  :stroke-width='linkWidth'
                  :class='linkClass(link) + " curve"'>
            </path>
        </g>
        <g class="labels" id="node-labels" v-if="nodeLabels">
            <text class="node-label" v-if="node.x" v-for="node in nodes"
                  :x='node.x + (getNodeSize(node) / 8) + (fontSize/2)'
                  :y='node.y + labelOffset.y'
                  :font-size="fontSize"
                  :stroke-width='fontSize / 4'
                  :class='nodeLabelClass(node)'>
                {{ node.name }}
            </text>
        </g>
        <g class="weight-labels" id="node-weight" v-if="nodeWeight">
            <text class="node-weight-label" v-if="node.x" v-for="node in nodes"
                  :x='node.x - ((getNodeSize(node) / 4) + (fontSize / 2))'
                  :y='node.y + labelOffset.y'
                  :font-size="fontSize"
                  :stroke-width='fontSize / 4'>
                {{ node.description }}
            </text>
        </g>
    </svg>
</template>
<script>
import svgExport from '../lib/svgExport.js'
import {event as d3Event} from 'd3-selection';
import {zoom as d3Zoom} from 'd3-zoom';
import {drag as d3Drag} from 'd3-drag';
import {select as d3Select} from 'd3-selection';

export default {
    name: 'svg-renderer',
    props: ['size',
        'nodes',
        'noNodes',
        'selected',
        'linksSelected',
        'links',
        'nodeSize',
        'padding',
        'fontSize',
        'strLinks',
        'linkWidth',
        'nodeLabels',
        'nodeWeight',
        'starMode',
        'linkLabels',
        'labelOffset',
        'nodeSym'],

    computed: {
        nodeSvg() {
            if (this.nodeSym) {
                return svgExport.toObject(this.nodeSym)
            }
            return null
        }
    },
    mounted() {
        this.size.h = this.size.h - 110;
        const zoom = d3Zoom()
            .scaleExtent([0.01, 20]) //zoom limit
            .on('zoom', () => {
                d3Select('g').style('stroke-width', `${1.5 / d3Event.transform.k}px`);
                d3Select('g').attr('transform', d3Event.transform); // updated for d3 v4
                d3Select('#l-links').style('stroke-width', `${1.5 / d3Event.transform.k}px`);
                d3Select('#l-links').attr('transform', d3Event.transform); // updated for d3 v4
                d3Select('#node-weight').style('stroke-width', `${1.5 / d3Event.transform.k}px`);
                d3Select('#node-weight').attr('transform', d3Event.transform); // updated for d3 v4
                d3Select('#node-labels').style('stroke-width', `${1.5 / d3Event.transform.k}px`);
                d3Select('#node-labels').attr('transform', d3Event.transform); // updated for d3 v4
            });
        d3Select('#network-graph').call(zoom)
            .append('svg:g')
            .attr('transform', 'translate(100,50) scale(.5,.5)');
    },
    methods: {
        /**
         * get node size
         *
         * @param {object} node
         * @param {string} side
         * @return {number}
         */
        getNodeSize(node, side) {
            let size = node._size || this.nodeSize;
            if (side) size = node['_' + side] || size;
            return size;
        },
        /**
         * call when mousedown fires
         *
         * @param {object} event
         * @param {number} key
         * @return {void}
         */
        mouseDown(event, key) {
            event.stopImmediatePropagation();
            this.emit("dragStart", [event, key])
        },
        /**
         * icon for node
         *
         * @param {object} node
         * @return {object}
         */
        svgIcon(node) {
            return node.svgObj || this.nodeSvg;
        }
        ,
        /**
         *  emit an event to parent component
         *
         * @param {string} e
         * @param {array|object|string|number} args
         * @return {void}
         */
        emit(e, args) {
            this.$emit('action', e, args);
        }
        ,
        /**
         * get screen shot from svg
         *
         * @param {function} cb
         * @return {void}
         */
        svgScreenShot(cb, toSvg, background, allCss) {
            let svg = svgExport.export(this.$refs.svg, allCss);
            if (!toSvg) {
                if (!background) background = this.searchBackground();
                let canvas = svgExport.makeCanvas(this.size.w, this.size.h, background);
                svgExport.svgToImg(svg, canvas, (err, img) => {
                    if (err) cb(err);
                    else cb(null, img);
                })
            } else {
                cb(null, svgExport.save(svg));
            }
        }
        ,
        /**
         *  class of a link
         *
         * @param {object} link
         * @return {string}
         */
        linkClass(link) {
            let cssClass = '';
            if (Object.getOwnPropertyNames(this.linksSelected).length > 1) {
                if (!this.linksSelected.hasOwnProperty(link.id)) {
                    cssClass += 'fadeLink dashLine';
                }
            } else {
                cssClass += 'link dashLine';
                if (link._linkClass) {
                    cssClass += (' ' + link._linkClass);
                }
            }
            if (this.linksSelected.hasOwnProperty(link.id)) {
                cssClass += 'link selected selectedLink dashLine'
            }
            return cssClass
        },
        /**
         *  class of a node label
         *
         * @param {object} node
         * @return {string}
         */
        nodeLabelClass(node){
            let cssClass = '';
            if (Object.getOwnPropertyNames(this.selected).length > 1 || Object.getOwnPropertyNames(this.linksSelected).length > 1) {
                if (!this.selected[node.id]) {
                    cssClass += 'fadeNodeLabel';
                }
            } else {
                cssClass = node._labelClass || '';
                cssClass += ' nodeLabel';
            }
            if (Object.getOwnPropertyNames(this.linksSelected).length > 1) {
                if (this.linksSelected[Object.keys(this.linksSelected)[0]].tid === node.id || this.linksSelected[Object.keys(this.linksSelected)[0]].sid === node.id) {
                    cssClass += 'selected selectedNodeLabel';
                }
            }
            if (this.selected[node.id]) {
                cssClass += 'selected selectedNodeLabel';
            }
            if (node.fx || node.fy) {
                cssClass += ' pinned';
            }
            return cssClass
        }
        ,
        /**
         *  create a curve for a link
         *
         * @param {object} link
         * @return {string}
         */
        curve(link) {
            let d = {
                M: [link.source.x, link.source.y],
                Q: [link.source.x, link.target.y],
                X: [link.target.x, link.target.y]
            };
            if (!link.source.x || !link.source.y || !link.target.x || !link.target.y) {
                d = {
                    M: [0, 0],
                    Q: [0, 0],
                    X: [0, 0]
                };
            }
            if (link.sid === link.tid) {
                d.X = [d.M[0] + 1 + ',' + d.M[1]];
                return "M " + d.M + " A 15,10 -90 1,1 " + d.X;
            } else {
                return 'M ' + d.M + ' Q ' + d.Q.join(' ') + ' ' + d.X;
            }
        }
        ,
        /**
         *  style of a node
         *
         * @param {object} node
         * @return {string}
         */
        nodeStyle(node) {
            return (node._color) ? 'fill: ' + node._color : ''
        }
        ,
        /**
         * get class of a node
         *
         * @param {object} node
         * @return {string}
         */
        nodeClass(node) {
            let cssClass = '';
            if (Object.getOwnPropertyNames(this.selected).length > 1 || Object.getOwnPropertyNames(this.linksSelected).length > 1) {
                if (!this.selected[node.id]) {
                    cssClass += 'fadeNode';
                }
            } else {
                cssClass = node._cssClass || '';
                cssClass += ' node';
            }
            if (Object.getOwnPropertyNames(this.linksSelected).length > 1) {
                if (this.linksSelected[Object.keys(this.linksSelected)[0]].tid === node.id || this.linksSelected[Object.keys(this.linksSelected)[0]].sid === node.id) {
                    cssClass += 'selected selectedNode';
                }
            }
            if (this.selected[node.id]) {
                cssClass += 'selected selectedNode';
            }
            if (node.fx || node.fy) {
                cssClass += ' pinned';
            }
            return cssClass
        }
        ,
        /**
         *  search for background
         *
         * @return {string}
         */
        searchBackground() {
            let vm = this;
            while (vm.$parent) {
                let style = window.getComputedStyle(vm.$el);
                let background = style.getPropertyValue('background-color');
                let rgb = background.replace(/[^\d,]/g, '').split(',');
                let sum = rgb.reduce((a, b) => parseInt(a) + parseInt(b), 0);
                if (sum > 0) return background;
                vm = vm.$parent;
            }
            return 'white'
        }
        ,
        /**
         * make a svg symbol node
         *
         * @return {*}
         */
        spriteSymbol() {
            let svg = this.nodeSym;
            if (svg) {
                return svgExport.toSymbol(svg);
            }
        }
    }
}
</script>
<style>
</style>