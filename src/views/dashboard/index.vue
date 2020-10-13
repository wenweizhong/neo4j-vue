<template>
  <!-- 显示知识图谱所有节点-->
  <div class="container">
  </div>
  <!-- 显示节点信息面板-->
</template>

<script>
  import * as d3 from 'd3'
  import axios from 'axios'
export default {
  name: 'Dashboard',
  data () {
    return {
      width: 800,
      height: 500,
      linksName: [],
      colorList:['red', 'blue', "red"],
      testGraph:{
        "nodes": [
          {"id": "Myriel", "group": 1},
          {"id": "OldMan", "group": 2},
          {"id": "NewMan", "group": 1},
          {"id": "WWW", "group":1 }
      ],
        "links": [
          {"source": "OldMan", "target": "Myriel", "value": 1, "relationship" : "ACTED_IN"},
          {"source": "OldMan", "target": "WWW", "value": 3, "relationship" : "ACTED_IN"},
          {"source": "OldMan", "target": "NewMan", "value": 2, "relationship" : "ACTED_IN"},
      ]},
    }
  },
  mounted() {
    //钩子函数支持链式编程
    axios.get("./assets/data.json").then(res => (this.testGraph["nodes"]=res.data["nodes"]))
    console.log(this.testGraph)
    this.initGraph(this.testGraph)
  },
  methods:{
    initGraph(data){
      var _this = this;
      const links = data.links.map(d => Object.create(d));
      const nodes = data.nodes.map(d => Object.create(d));

      const simulation = d3.forceSimulation(nodes)
         .force("link", d3.forceLink(links).id(d => d.id).distance(150))
         .force("collide", d3.forceCollide().radius(()=>30))//碰撞半径
         .force("charge", d3.forceManyBody().strength(0))//两个节点的距离
         .force("center", d3.forceCenter(_this.width / 2, _this.height / 2));

      const svg = d3.select(".container")
        .append("svg")
        .attr("viewBox", [0, 0, _this.width, _this.height])
        .call(d3.zoom().on("zoom", function(){
          g.attr("transform", d3.event.transform)
        }))

      const marker = svg.append("marker")
        .attr("id", "direction")
        .attr("stroke-width", 2)
        .attr("markerUnits", "strokeWidth")
        .attr("markerUnits", "userSpaceOnUse")
        .attr("viewBox", "0 -5 10 10")//坐标系的区域
        .attr("refX",35)//箭头坐标
        .attr("refY", 0)
        .attr("markerWidth", 12)//标识的大小
        .attr("markerHeight", 12)
        .attr("orient", "auto")//绘制方向，可设定为：auto（自动确认方向）和 角度值
        .append("path")
        .attr("d", "M 0 -5 L 10 0 L 0 5")//箭头的路径
        .attr('fill','gray');//箭头颜色



      const g = svg.append("g")

      const link = g.append("g")
       .attr("stroke", "#999")
       .attr("stroke-opacity", 0.6)
       .attr("marker-end", "url(#direction)")
       .selectAll("path")
       .data(links, function(d){
          if(typeof(d.source) === 'object'){
            return d.source.id + "_" + d.relationship + "_" + d.target.id
          }else{
            return d.source + "_" + d.relationship + "_" + d.target
          }
       })
       .join("path")
       .attr("stroke-width", d => Math.sqrt(d.value))
       .attr("id", function(d){
          if(typeof(d.source) === 'object'){
            return d.source.id + "_" + d.relationship + "_" + d.target.id
          }else{
            return d.source + "_" + d.relationship + "_" + d.target
          }
        });

      _this.linksName = g.append("g")
         .selectAll("text")
         .data(links, function(d){
            if(typeof(d.source) === 'object'){
              return d => d.source.id + "_" + d.relationship + "_" + d.target.id
            }else{
              return d => d.source + "_" + d.relationship + "_" + d.target
            }
          })
         .join("text")
         .style("fill", "white")
         .style("font-size", "10px")
         .style("font-weight", "bold")
         .append("textPath")
         //两个节点连接需要指向id属性，不能用对象  不然会出Bug
         .attr('xlink:href', d => "#" + d.source.id + "_" + d.relationship + "_" + d.target.id)
         .attr('startOffset', '40%')
         .text(function(d){
           return d.relationship
       });

      const node = g.append("g")
       .selectAll("circle")
       .data(nodes)
       .join("circle")
       .attr("r", 30)
       .attr("class", "node")
       .attr("fill", _this.color)
       .on("click", _this.queryTest) //点击该点会调用查询该点信息的方法
       .call(_this.drag(simulation))

      node.append("title")
         .text(d => d.id);


      const nodeNameText = g.append("g")
        .selectAll("text")
        .data(nodes)
        .join("text")
        .attr("dx", -25)
        .attr("dy", 50)
        .attr("class", "node-name")
        .attr("fill", "white")
        .text(function(d){
          return d.id;
        })

      simulation.on("tick", () => {
        link
        //path连接方式
          .attr("d", d => "M" + d.source.x + " " + d.source.y + "L" + d.target.x + " " + d.target.y);

        node
          .attr("cx", d => d.x)
          .attr("cy", d => d.y);

        nodeNameText
          .attr("x", d => d.x)
          .attr("y", d => d.y);
      });
    },

    color(d){
      return this.colorList[d.group];
    },

    drag(simulation){
      function dragstarted(event) {
          if (!event.active) simulation.alphaTarget(0.01).restart();
          event.fx = event.x;
          event.fy = event.y;
      }

      function dragged(event) {
        event.fx = d3.event.x;
        event.fy = d3.event.y;
      }

      function dragended(event) {
        if (!event.active) simulation.alphaTarget(0);
        event.fx = null;
        event.fy = null;
      }

      return d3.drag()
            .on("start", dragstarted)
            .on("drag", dragged)
            .on("end", dragended);
    },

    queryTest(d){
      /* 用来显示每个节点的详细信息*/
      console.log(d.id)
    },

    getGraphData(){
      var _this = this
      this.axios.get("person/all")
      .then(function (response){
        _this.testGraph["nodes"] = [responese.data]
        _this.initGraph(_this.testGraph)
      })
      .catch(function (error){
        console.log(error)
      })
    }

  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.container{
  width: 50rem;
  height: 31.25rem;
  border: 1px solid #2C3E50;
  border-radius: 0.5rem;
  margin-top: 2.5rem;
  margin-left: auto;
  margin-right: auto;
  background: #154360 repeating-linear-gradient(30deg,
  hsla(0, 0%, 100%, .1), hsla(0, 0%, 100%, .1) 0.9375rem,
  transparent 0, transparent 1.875rem);
}
</style>
<style>
.node{
  stroke: #fff;
  stroke-width: 1;
  cursor: pointer;
}
.node:hover{
  stroke-width: 5;
}
.nodeName{
  fill:white;
}
</style>
