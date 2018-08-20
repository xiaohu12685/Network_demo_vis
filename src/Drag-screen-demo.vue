<template>
  <div id="app">
      <div class="menu">
          <button @click="ClickSwitchState('1')" :class="{'active':typer=='1'}" class="item_btn">添加</button>
          <button @click="ClickSwitchState('2')" :class="{'active':typer=='2'}" class="item_btn">连线</button>
          <button @click="ClickSwitchState('3')" :class="{'active':typer=='3'}" class="item_btn">删除</button>
      </div>
      <div id="mynetwork" class="canvas"></div>
      <div class="data">
          <p>nodes:{{nodes}}</p>
          <p>edges:{{edges}}</p>
      </div>
  </div>
</template>

<script>
import vis from './assets/lib/vis.js';
import './assets/lib/vis.css';
export default {
  data(){
    return {
      nodes:[],
      edges:[],
      lengt:[],
      typer:false
    }
  },
  methods: {
			cavs(){
        // 创建节点数组
        var nodes = new vis.DataSet(this.nodes);
        // 创建关系数组
        var edges = new vis.DataSet(this.edges);
        // 创建一个网络
        var container = document.getElementById('mynetwork');
        // vis数据
        var data = {
            nodes: nodes,
            edges: edges
        };
        var options = {
            // autoResize: false,
            nodes: {
                font: {
                    color: '#fff'
                },
                borderWidth: 1,
                borderWidthSelected: 3,
                color: {
                    border: 'rgba(1,1,1,0)',
                    highlight: {
                        border: '#ffffff',
                    },
                    hover: {
                        border: '#ffffff',
                    }
                },
                shapeProperties: {
                    useBorderWithImage: true
                }
            },
            edges: {
                arrows: 'to',
                color: {
                    color: '#4C5967',
                    highlight: '#4C5967',
                    hover: '#4C5967'
                },
                length: 200,
                width: 1,
                // selectionWidth: 0,
                // hoverWidth: 0,
                font: {
                    size: 12,
                    strokeWidth: 0,
                },
                smooth: {
                    enabled: false,
                    type: 'cubicBezier',
                    roundness: 0.8
                }
            },
            // 关闭物理引擎
            physics: {
                enabled: false
            },
            layout: {
                randomSeed: 9,
            },
            interaction: {
                dragNodes: true, //是否能拖动节点
                dragView: true, //是否能拖动画布
                hover: true, //鼠标移过后加粗该节点和连接线
                multiselect: true, //按 ctrl 多选
                selectable: true, //是否可以点击选择
                selectConnectedEdges: true, //选择节点后是否显示连接线
                hoverConnectedEdges: true, //鼠标滑动节点后是否显示连接线
                zoomView: true //是否能缩放画布
            },
            manipulation: {
                enabled: false,
                addEdge: (edge, callback) => {
                    this.isClosedLine(edge);
                }
            }
        };
        // 初始化网络
        this.network = new vis.Network(container, data, options);
        let that = this;
        this.network.on('click', function (params) {
            if(params.nodes.length == 0){
                if (that.typer == '1') {
                    const center = params.pointer.canvas;
                    let nodes = that.nodes;
                    const id = that.processUUID();
                    let json = {};
                    json.id = id;
                    json.shape = 'box';
                    json.x = center.x;
                    json.y = center.y;
                    json.data = ['一些数据什么的'];
                    json.label = `节点 ${(nodes.length*1)+1}`;
                    // json.name = ((nodes.length*1)+1)*1111;
                    nodes.push(json);
                    that.nodes = nodes;
                    that.lengt = [];
                    that.cavs();
                    that.typer = false;
                    return;
                }
            }
            // that.network.addNodeMode();
        });
        this.network.on("selectNode", function (params) {
            if(that.typer == '3'){
                let data = that.nodes;
                data.map((item,index)=>{
                    if(item.id == params.nodes[0]){
                        data.splice(index,1);
                    }
                });
                that.nodes = data;
                that.cavs();
                // that.network.deleteSelected();
                that.typer = false;
            }
        });
        this.network.on('dragEnd', function (params) {
            if(params.nodes.length == 0){
                console.log('拖动的是空白处');
            }else{
                let nodes = that.nodes;
                nodes.forEach(item => {
                    if(item.id == params.nodes[0]){
                        item.x = params.pointer.canvas.x;
                        item.y = params.pointer.canvas.y;
                    }
                });
                that.nodes = nodes
            }
        });
        this.network.moveTo({//固定布局位置
            position:{
                x:0,
                y:0
            },
        });
    },
    ClickSwitchState(e){
      if(e == '2'){
        this.network.addEdgeMode();
      }
      this.typer = this.typer==e?false:e;
    },
    processUUID(){
        const urid = 'yxx-xyx-xxy-xxxx'.replace(/[xy]/g, function (a){
            const b = Math.random()*13 | 0 , c = a == 'x'? b : (b & 0x3 | 0x8);
            return c.toString(16);
        })
        return urid;
    },
    isClosedLine(edge){
        if (edge.from == edge.to) {
            return false;
        } else {
            const iterator = (key, aimKey) => {
                let edges = this.edges;
                edges.push({from: key, to: aimKey});
                this.edges = edges,
                this.cavs();
                this.typer = false;
            };
            iterator(edge.from, edge.to);
        }
    }
    
  },
  mounted(){
    this.cavs();
  }
}
</script>

<style>
    *{
      margin: 0;
      padding: 0;
    }
    .menu{
      width: 100%;
      padding: 20px 30%;
      box-sizing: border-box;
      display: flex;
      justify-content: space-around;
    }
    .item_btn{
      outline: 0;
      font-weight: 400;
      cursor: pointer;
      border: 1px solid transparent;
      user-select: none;
      padding: 5px 15px 6px;
      font-size: 12px;
      border-radius: 4px;
      transition: color .2s linear,background-color .2s linear,border .2s linear,box-shadow .2s linear;
      color: #515a6e;
      background-color: #fff;
      border-color: #dcdee2;
    }
    .item_btn:hover {
        color: #57a3f3;
        background-color: #fff;
        border-color: #57a3f3;
    }
    .active{
        color: #57a3f3;
        background-color: #fff;
        border-color: #57a3f3;
        box-shadow: 0 0 0 2px rgba(45,140,240,.2);
    }
    .item_btn:not([disabled]):hover {
        text-decoration: none;
    }
    [type=button] {
        -webkit-appearance: button;
    }

    .canvas{
      width: 100%;
      height: 800px;
      background: #808695;
    }
    .data{
        padding-top: 10px;
        font-size: 14px;
        color: #808695;
    }
    .data p:first-child{
        margin-bottom: 5px;
    }
</style>
