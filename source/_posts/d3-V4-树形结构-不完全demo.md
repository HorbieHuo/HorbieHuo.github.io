---
title: d3 V4 树形结构 不完全demo
date: 2017-07-26 16:46:11
tags: d3
---
* 首先贴个小图
![d3_tree](http://oh14x29vm.bkt.clouddn.com/d3_tree.jpg "d3_tree")

* 下面是显示上面图片的部分代码
```html
<script src="/admin/lib/d3/d3.v4.min.js"></script>
<style>
    .node circle {
        fill: #fff;
        stroke: steelblue;
        stroke-width: 1.5px;
    }

    .node {
        font: 12px sans-serif;
    }

    .link {
        fill: none;
        stroke: #ccc;
        stroke-width: 1.5px;
    }
</style>
<body>
    <div id="auth_body"></div>
</body>

<script type="text/javascript">
    var width = 500,
        height = 500;

    var tree = d3.tree()
        .size([width, height - 200])
        .separation(function (a, b) {
            return (a.parent == b.parent ? 1 : 2);
        });

    var svg = d3.select("#auth_body").append("svg")
        .attr("width", width)
        .attr("height", height)
        .append("g")
        .attr("transform", "translate(40,0)");


    var root = {
        "name": "中国",
        "children": [{
                "name": "浙江",
                "children": [{
                        "name": "杭州"
                    },
                    {
                        "name": "宁波"
                    },
                    {
                        "name": "温州"
                    },
                    {
                        "name": "绍兴"
                    }
                ]
            },

            {
                "name": "广西",
                "children": [{
                        "name": "桂林",
                        "children": [{
                                "name": "秀峰区"
                            },
                            {
                                "name": "叠彩区"
                            },
                            {
                                "name": "象山区"
                            },
                            {
                                "name": "七星区"
                            }
                        ]
                    },
                    {
                        "name": "南宁"
                    },
                    {
                        "name": "柳州"
                    },
                    {
                        "name": "防城港"
                    }
                ]
            },

            {
                "name": "黑龙江",
                "children": [{
                        "name": "哈尔滨"
                    },
                    {
                        "name": "齐齐哈尔"
                    },
                    {
                        "name": "牡丹江"
                    },
                    {
                        "name": "大庆"
                    }
                ]
            },

            {
                "name": "新疆",
                "children": [{
                        "name": "乌鲁木齐"
                    },
                    {
                        "name": "克拉玛依"
                    },
                    {
                        "name": "吐鲁番"
                    },
                    {
                        "name": "哈密"
                    }
                ]
            }
        ]
    }
    var hierarchy = d3.hierarchy(root);
    tree(hierarchy);

    function diagonal(d) {
        if (d.parent === hierarchy.descendants[0]) {
            return "M" + d.y + "," + d.x +
                " " + d.parent.y + "," + d.parent.x
        } else {
            return "M" + d.y + "," + d.x +
                "C" + (d.parent.y + 100) + "," + d.x +
                " " + (d.parent.y + 100) + "," + d.parent.x +
                " " + d.parent.y + "," + d.parent.x;
        }
    }

    var link = svg.selectAll(".link")
        .data(hierarchy.descendants().slice(1))
        .enter()
        .append("path")
        .attr("class", "link")
        .attr("d", diagonal);

    var node = svg.selectAll(".node")
        .data(hierarchy.descendants())
        .enter()
        .append("g")
        .attr("class", "node")
        .attr("transform", function (d) {
            return "translate(" + d.y + "," + d.x + ")";
        })

    node.append("circle")
        .attr("r", 4.5);

    node.append("text")
        .attr("dx", function (d) {
            return d.children ? -8 : 8;
        })
        .attr("dy", 3)
        .style("text-anchor", function (d) {
            return d.children ? "end" : "start";
        })
        .text(function (d) {
            return d.data.name;
        });
</script>
```