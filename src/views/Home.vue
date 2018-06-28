<template>
    <my-page title="数学图表" :page="page">
        <div class="graph-box">
            <div id="jxgbox" class="graph">
                <svg class="svg"></svg>
            </div>
            <!-- <ui-icon-button class="float-btn setting-icon" icon="settings" /> -->
            <ui-icon-button class="float-btn setting-o" icon="my_location" @click="zoom100" />
            <ui-icon-button class="float-btn zoom-in" icon="zoom_in" @click="zoomIn" />
            <ui-icon-button class="float-btn zoom-out" icon="zoom_out" @click="zoomOut" />
            <ui-icon-button class="float-btn help" icon="help_outline" @click="help" />
        </div>
        <div class="tool-box">
            <ui-tabs :value="activeTab" @change="handleTabChange">
                <ui-tab value="tab1" title="工具"/>
                <ui-tab value="tab2" title="形状"/>
                <ui-tab value="tab3" title="预设"/>
            </ui-tabs>
            <div class="body" v-if="activeTab === 'tab1'">
                <!-- <div class="input-box">
                    <input class="input" v-model="input" />
                    <ui-icon-button icon="add" @click="add" />
                </div> -->
                <ui-sub-header class="sub-header">基本工具</ui-sub-header>
                <ul class="mode-list">
                    <li class="item" :title="m.title" v-for="m in modes" @click="selectMode(m)">
                        <img class="img" :src="'/static/img/' + m.icon">
                    </li>
                </ul>
                <ui-sub-header class="sub-header">线</ui-sub-header>
                <ul class="mode-list">
                    <li class="item" :title="m.title" v-for="m in modes2" @click="selectMode(m)">
                        <img class="img" :src="'/static/img/' + m.icon">
                    </li>
                </ul>
                <ui-sub-header class="sub-header">圆</ui-sub-header>
                <ul class="mode-list">
                    <li class="item" :title="m.title" v-for="m in modes4" @click="selectMode(m)">
                        <img class="img" :src="'/static/img/' + m.icon">
                    </li>
                </ul>
                <ui-sub-header class="sub-header">结构</ui-sub-header>
                <ul class="mode-list">
                    <li class="item" :title="m.title" v-for="m in modes5" @click="selectMode(m)">
                        <img class="img" :src="'/static/img/' + m.icon">
                    </li>
                </ul>
                <ui-sub-header class="sub-header">其他</ui-sub-header>
                <ul class="mode-list">
                    <li class="item" :title="m.title" v-for="m in otherModes" @click="selectMode(m)">
                        <img class="img" :src="'/static/img/' + m.icon">
                    </li>
                </ul>
            </div>
            <div class="body no-padding" v-if="activeTab === 'tab2'">
                <ul class="shape-list">
                    <div class="empty" v-if="!elems.length">
                        暂无图形
                    </div>
                    <li class="item" v-for="elem in elems">
                        {{ elem.text }}
                        <a href="" @click.prevent="removeElem(elem)">删除</a>
                    </li>
                </ul>
            </div>
            <div class="body" v-if="activeTab === 'tab3'">
                <div class="form-item">
                    <div class="btns">
                        <ui-raised-button class="btn" label="抛物线" @click="add1" />
                        <ui-raised-button class="btn" label="三点共圆" @click="add3PointCircle" />
                        <ui-raised-button class="btn" label="弧线" @click="addArc" />
                        <ui-raised-button class="btn" label="马蹄形" @click="addCircumcircleSector" />
                        <ui-raised-button class="btn" label="样条曲线" @click="addSpline" />
                        <ui-raised-button class="btn" label="线段中点" @click="addMidpoint2" />
                    </div>
                </div>
            </div>
        </div>
        <ui-dialog dialogClass="text-dialog" :open="textDialogVisible" title="文本" @close="toggleTextDialog">
            <ui-text-field v-model="text" />
            <ui-flat-button slot="actions" @click="toggleTextDialog" primary label="取消"/>
            <ui-flat-button slot="actions" primary @click="textDialogOk" label="确定"/>
        </ui-dialog>
        <ui-dialog dialogClass="text-dialog" :open="radiusDialogVisible" title="半径" @close="toggleRadiusDialog">
            <ui-text-field v-model.number="radius" />
            <ui-flat-button slot="actions" @click="toggleRadiusDialog" primary label="取消"/>
            <ui-flat-button slot="actions" primary @click="radiusDialogOk" label="确定"/>
        </ui-dialog>
    </my-page>
</template>

<script>
    /* eslint-disable */
    export default {
        data () {
            return {
                textDialogVisible: false,
                radiusDialogVisible: false,
                mode: 'select',
                modes: [
                    {
                        id: '1',
                        name: 'select',
                        title: '选择',
                        icon: 'mode_select.svg'
                    },
                    {
                        id: '2',
                        name: 'point',
                        title: '点',
                        icon: 'mode_point.svg'
                    },
                    {
                        id: '3',
                        name: 'segment',
                        title: '线段',
                        icon: 'mode_segment.svg'
                    },
                    {
                        id: '4',
                        name: 'line',
                        title: '直线',
                        icon: 'mode_line.svg'
                    },
                    {
                        id: '4',
                        name: 'circle',
                        title: '圆',
                        icon: 'mode_circle.svg'
                    }
                ],
                modes2: [
                    {
                        id: '3',
                        name: 'segment',
                        title: '线段',
                        icon: 'mode_segment.svg'
                    },
                    {
                        id: '3',
                        name: 'segment_fixed_length',
                        title: '线段（定长）',
                        icon: 'mode_segment_fixed_length.svg'
                    },
                    {
                        id: '4',
                        name: 'line',
                        title: '直线',
                        icon: 'mode_line.svg'
                    },
                ],
                otherModes: [
                    {
                        id: '3',
                        name: 'polygon',
                        title: '多边形',
                        icon: 'mode_polygon.svg'
                    },
                    {
                        id: '3',
                        name: 'text',
                        title: '文本',
                        icon: 'mode_text.svg'
                    },
                ],
                modes4: [
                    {
                        id: '4',
                        name: 'circle',
                        title: '圆',
                        icon: 'mode_circle.svg'
                    },
                    {
                        id: '4',
                        name: 'mode_circle_fixed_radius',
                        title: '圆（固定半径）',
                        icon: 'mode_circle_fixed_radius.svg'
                    },
                    {
                        id: '3',
                        name: 'semicircle',
                        title: '半圆',
                        icon: 'mode_semicircle.svg'
                    },
                    {
                        id: '3',
                        name: 'sector',
                        title: '扇形',
                        icon: 'mode_sector.svg'
                    },
                    {
                        id: '3',
                        name: 'ellipse',
                        title: '椭圆',
                        icon: 'mode_ellipse.svg'
                    },
                ],
                modes5: [
                    {
                        id: '3',
                        name: 'midpoint',
                        title: '中点',
                        icon: 'mode_midpoint.svg'
                    },
                    {
                        id: '3',
                        name: 'parallel',
                        title: '平行线',
                        icon: 'mode_parallel.svg'
                    },
                    {
                        id: '3',
                        name: 'perpendicular',
                        title: '垂线',
                        icon: 'mode_perpendicular.svg'
                    },
                    {
                        id: '3',
                        name: 'angle',
                        title: '角度',
                        icon: 'mode_angle.svg'
                    },
                ],
                activeTab: 'tab1',
                text: '',
                input: '(1, 2)',
                radius: 4,
                elems: [],
                page: {
                    menu: [
                        {
                            type: 'icon',
                            icon: 'help',
                            to: '/help'
                        }
                    ]
                }
            }
        },
        mounted() {
            this.init()
        },
        methods: {
            toggleRadiusDialog() {
                this.radiusDialogVisible = !this.radiusDialogVisible
            },
            radiusDialogOk() {
                if (!this.radius) {
                    this.$message({
                        type: 'danger',
                        text: '请输入半径'
                    })
                    return
                }
                this.addCircle()
                this.radiusDialogVisible = false
            },
            toggleTextDialog() {
                this.textDialogVisible = !this.textDialogVisible
            },
            textDialogOk() {
                if (!this.text) {
                    this.$message({
                        type: 'danger',
                        text: '请输入文本'
                    })
                    return
                }
                this.addText()
                this.textDialogVisible = false
            },
            selectMode(mode) {
                console.log(mode)
                switch (mode.name) {
                    case 'point':
                        this.addPoint()
                        break
                    case 'segment':
                        this.addSegment()
                        break
                    case 'segment_fixed_length':
                        this.addSegment2()
                        break    
                    case 'line':
                        this.addLine()
                        break
                    case 'circle':
                        this.addCircle2()
                        break
                    case 'semicircle':
                        this.addCemicircle()
                        break
                    case 'polygon':
                        this.addRect()
                        break
                    case 'sector':
                        this.addSector()
                        break
                    case 'text':
                        this.toggleTextDialog()
                        break
                    case 'mode_circle_fixed_radius':
                        this.toggleRadiusDialog()
                        break
                    case 'parallel':
                        this.addParallel()
                        break
                    case 'ellipse':
                        this.addEllipse()
                        break
                    case 'angle':
                        this.addAngle()
                        break
                    case 'perpendicular':
                        this.addPerpendicular()
                        break
                    case 'midpoint':
                        this.addMidpoint()
                        break
                }
            },
            handleTabChange (val) {
                this.activeTab = val
            },
            init() {
                JXG.Options.grid.gridOpacity = '70';
                JXG.Options.grid.gridDash = false;
                var board = JXG.JSXGraph.initBoard('jxgbox', {
                    mode: JXG.Board.BOARD_MODE_DRAG,
                    boundingbox: [-10.9, 8.55, 10.9, -8.55],
                    showCopyright: false,
                    grid: true,
                    snapToGrid: true,
                    keepaspectratio: true,
                    selection: {
                        enabled: true,
                        name: 'selectionPolygon',
                        needShift: false,  // mouse selection needs pressing of the shift key
                        needCtrl: true,    // mouse selection needs pressing of the shift key
                        withLines: false,  // Selection polygon has border lines
                        vertices: {
                            visible: false
                        },
                        showClearTraces: true,
                        fillColor: '#ffff00',
                        visible: false      // Initial visibility. Should be set to false always
                    }
                })
                this.board = board
                this.board.showInfobox(false)

                var qr = [];
                

                // var p1 = board.create('point', [4.5, 2.0]);
                // var p2 = board.create('point', [1.0, 1.0]);
                // var l1 = board.create('arrow', [p1, p2]);

                // A
                // qr[0] = board.create('point', [2, 2], {
                //     fixed: false,
                //     style: 3,
                //     strokeColor: '#ff00ff',
                //     fillColor: '#ff00ff'
                // });
                // qr[1] = board.create('point', [4, 0], {
                //     fixed: false,
                //     style: 3,
                //     strokeColor: '#ff00ff',
                //     fillColor: '#ff00ff' });
                // var p3 = board.create('point', [7, 4], {
                //     fixed: false,
                //     style: 3,
                //     strokeColor: '#ff00ff',
                //     fillColor: '#ff00ff'
                // });
                // var lim1 = board.create('line', [qr[0], qr[1]], {
                //     strokeWidth: 2,
                //     strokeColor: '#ff00ff'
                // });
                    
                // var reflp3 = board.create('reflection', [lim1, p3], { visible: false }); var perp1 = board.create('perpendicular', [lim1, p3], { strokeWidth: 1, strokeColor: '#2222ff', dash: 2 }); 
                
                // var perp2 = board.create('perpendicular', [lim1, reflp3], { strokeWidth: 1, strokeColor: '#2222ff', dash: 2 }); 
                
                // var p4 = board.create('perpendicularpoint', [lim1, p3], { visible: false }); 
                
                // let el1 = board.create('text', [-1, function (x) { return Math.max(-8, (qr[1].Y() - qr[1].X() * (qr[0].Y() - qr[1].Y()) / (qr[0].X() - qr[1].X()))) }, function (x) { return "<p class='gborder'>斜率 <i>m</i><sub>1</sub> = " + lim1.getSlope().toFixed(2); }]);
                
                // let midpt = board.create('midpoint', [p3, p4], { visible: false })
                
                // var el2 = board.create('text', [function (x) { return midpt.X() - 1 }, function (x) { return midpt.Y() }, function (x) { return "<p class='gborder'>斜率 <i>m</i><sub>2</sub> = " + (-1 / (lim1.getSlope())).toFixed(2); }]);
                
                
                var fp = board.create('point', [0, 0], { fixed: true, visible: false });
                var l1 = board.create('line', [fp, [5, 0]], { strokeWidth: 1, strokeColor: '#5b8a9b', strokeOpacity: 0.8, lastArrow: true });
                
                var l2 = board.create('line', [fp, [0, 5]], { strokeWidth: 1, strokeColor: '#5b8a9b', strokeOpacity: 0.5, lastArrow: true });
                
                var pX0 = board.create('point', [0, function () { var bb = board.getBoundingBox(); return bb[3] + (bb[1] - bb[3]) * 0.07; }], { visible: false, withLabel: false }); var pX1 = board.create('point', [1, function () { var bb = board.getBoundingBox(); return bb[3] + (bb[1] - bb[3]) * 0.07; }], { visible: false, withLabel: false });
                var xaxis = board.create('axis', [pX0, pX1], { strokeWidth: 1, strokeColor: '#bfbfbf', lastArrow: false });
                pX0.type = JXG.OBJECT_TYPE_CAS; pX1.type = JXG.OBJECT_TYPE_CAS; 
                var pX2 = board.create('point', [function () { var bb = board.getBoundingBox(); return bb[0]; }, 0], { visible: false, withLabel: false });
                var pX3 = board.create('point', [function () { var bb = board.getBoundingBox(); return bb[0]; }, 1], { visible: false, withLabel: false }); var xaxis = board.create('axis', [pX2, pX3], { lastArrow: false }); pX2.type = JXG.OBJECT_TYPE_CAS; pX3.type = JXG.OBJECT_TYPE_CAS;
                var xaxisLabel = board.create('glider', [function () { var bb = board.getBoundingBox(); return bb[2] - (bb[2] - bb[0]) * 0.04; }, 0, l1], { strokeOpacity: 0, fillOpacity: 0, name: '<i>x</i>' }); 
                xaxisLabel.type = JXG.OBJECT_TYPE_CAS; 
                var yaxisLabel = board.create('glider', [0, function () { var bb = board.getBoundingBox(); return bb[1] - (bb[1] - bb[3]) * 0.05; }, l2], { strokeOpacity: 0, fillOpacity: 0, name: '<i>y</i>' }); 
                yaxisLabel.type = JXG.OBJECT_TYPE_CAS;
    
            },
            zoom100() {
                console.log(1)
                this.board.zoom100()
            },
            zoomOut() {
                this.board.zoomOut()
            },
            zoomIn() {
                this.board.zoomIn()
            },
            removeElem(elem) {
                let id = elem._id
                console.log(elem._id)
                elem._elem.remove()
                for (let i = 0; i < this.elems.length; i++) {
                    if (this.elems[i]._id === id) {
                        this.elems.splice(i, 1)
                        break
                    }
                }
            },
            add() {
                let m
                if (m = this.input.match(/\((\d),\s+(\d)\)/)) {
                    let x = parseFloat(m[1])
                    let y = parseFloat(m[2])
                    let p = this.board.create('point', [x, y], {
                        // fixed: false,
                        // style: 3,
                        // strokeColor: '#ff00ff',
                        // fillColor: '#ff00ff'
                        name: 'A'
                    })
                    console.log(p)
                    this.elems.push({
                        _id: p.id,
                        _elem: p,
                        type: 'point',
                        text: 'A'
                    })
                }
            },
            addPoint() {
                let p = this.board.create('point', [0, 0], {
                    // fixed: false,
                    // style: 3,
                    // strokeColor: '#ff00ff',
                    // fillColor: '#ff00ff'
                    // name: 'A'
                })
                console.log(p)
            },
            addCircle() {
                let p = this.board.create('point', [0, 0])
                this.board.create('circle', [p, this.radius])
            },
            addCircle2() {
                var p1 = this.board.create('point', [0, 0]),
                p2 = this.board.create('point', [2, 0]),
                c1 = this.board.create('circle', [p2, p1]);
            },
            addCemicircle() {
                var p1 = this.board.create('point', [4.5, 2.0]);
                var p2 = this.board.create('point', [1.0, 0.5]);

                var a = this.board.create('semicircle', [p1, p2]);
            },
            addSector() {
                // Create a sector out of three free points
                var p1 = this.board.create('point', [1.5, 5.0]),
                p2 = this.board.create('point', [1.0, 0.5]),
                p3 = this.board.create('point', [5.0, 3.0]),

                a = this.board.create('sector', [p1, p2, p3], {
                    hasInnerPoints: true
                });
            },
            addRect() {
                var p1 = this.board.create('point', [-4, 4]);
                var p2 = this.board.create('point', [4, 4]);
                var p3 = this.board.create('point', [4, -4]);
                var p4 = this.board.create('point', [-4, -4]);

                var pol = this.board.create('polygon', [p1, p2, p3, p4], {
                    hasInnerPoints: true
                })
            },
            addEllipse() {
                var A = this.board.create('point', [-1,4]);
                var B = this.board.create('point', [-1,-4]);
                var C = this.board.create('point', [1,1]);
                var el = this.board.create('ellipse',[A,B,C]);
            },
            addSegment() {
                var p1 = this.board.create('point', [4.5, 2.0]);
                var p2 = this.board.create('point', [1.0, 1.0]);
                var l1 = this.board.create('segment', [p1, p2]);
            },
            addSegment2() {
                var p3 = this.board.create('point', [4.0, 2.0]);
                var p4 = this.board.create('point', [1.0, 2.0]);
                var l2 = this.board.create('segment', [p3, p4, 3]);
            },
            addLine() {
                var p = this.board.create('point', [4.0, 2.0]);
                var p2 = this.board.create('point', [1.0, 2.0]);
                this.board.create('line', [p, p2], {
                    // strokeWidth: 1,
                    // strokeColor: '#5b8a9b',
                    // strokeOpacity: 0.8,
                });
            },
            addText() {
                var t1 = this.board.create('text', [0, 1, this.text || 'A'])
    //             var p = board.create('point',[0, 1]),
    // t = board.create('text',[0, -1,"Hello World"], {anchor: p});
            },
            addParallel() {
                var p1 = this.board.create('point', [0.0, 2.0]);
                var p2 = this.board.create('point', [2.0, 1.0]);
                var l1 = this.board.create('line', [p1, p2]);

                var p3 = this.board.create('point', [3.0, 3.0]);
                var pl1 = this.board.create('parallel', [l1, p3]);
            },
            addAngle() {
                var p1 = this.board.create('point', [-1, 4]),
                p2 = this.board.create('point', [4, 1]),
                q1 = this.board.create('point', [-2, -3]),
                q2 = this.board.create('point', [4,3]),

                li1 = this.board.create('line', [p1,p2], {strokeColor:'black', lastArrow:true}),
                li2 = this.board.create('line', [q1,q2], {lastArrow:true}),

                a1 = this.board.create('angle', [li1, li2, [5.5, 0], [4, 3]], { radius:1 })
            },
            addPerpendicular() {
                var p1 = this.board.create('point', [0.0, 2.0]);
                var p2 = this.board.create('point', [2.0, 1.0]);
                var l1 = this.board.create('line', [p1, p2]);

                var p3 = this.board.create('point', [3.0, 3.0]);
                var perp1 = this.board.create('perpendicularsegment', [l1, p3]);
            },
            add1() {
                var c1 = this.board.create('curve', [
                    function(t){
                        return t
                    },
                    x => {
                        return 1 * x * x
                    }
                ])
                var g1 = this.board.create('glider', [0.6, 1.2, c1]);
//   var t1 = board.create('tangent', [g1]);
            },
            add3PointCircle() {
                var p1 = this.board.create('point', [0.0, 2.0]);
                var p2 = this.board.create('point', [2.0, 1.0]);
                var p3 = this.board.create('point', [3.0, 3.0]);

                var cc1 = this.board.create('circumcircle', [p1, p2, p3]);
            },
            addArc() {
                let p1 = this.board.create('point', [2.0, 2.0]);
                let p2 = this.board.create('point', [1.0, 0.5]);
                let p3 = this.board.create('point', [3.5, 1.0]);
                let a = this.board.create('arc', [p1, p2, p3]);
            },
            addCircumcircleSector() {
                var p1 = this.board.create('point', [1.5, 5.0]),
                p2 = this.board.create('point', [1.0, 0.5]),
                p3 = this.board.create('point', [5.0, 3.0]),

                a = this.board.create('circumcirclesector', [p1, p2, p3], {
                    hasInnerPoints: true
                });
            },
            addSpline() {
                var p = [];
                p[0] = this.board.create('point', [-2,2], {size: 4, face: 'o'});
                p[1] = this.board.create('point', [0,-1], {size: 4, face: 'o'});
                p[2] = this.board.create('point', [2,0], {size: 4, face: 'o'});
                p[3] = this.board.create('point', [4,1], {size: 4, face: 'o'});

                var c = this.board.create('spline', p, {strokeWidth:3});
            },
            addMidpoint() {
                var p1 = this.board.create('point', [0.0, 2.0]);
                var p2 = this.board.create('point', [2.0, 1.0]);
                var mp1 = this.board.create('midpoint', [p1, p2]);
                // var l1 = this.board.create('segment', [[0.0, 3.0], [3.0, 3.0]]);
            },
            addMidpoint2() {
                var p1 = this.board.create('point', [0.0, 2.0]);
                var p2 = this.board.create('point', [2.0, 1.0]);
                var l1 = this.board.create('segment', [p1, p2]);

                var mp2 = this.board.create('midpoint', [l1]);
            },
            help() {
                this.$router.push('/help')
            }
        }
    }
</script>

<style lang="scss" scoped>

</style>
