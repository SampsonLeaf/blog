<script setup>
import { ref, onMounted } from 'vue'
import { Graph } from '@antv/g6'

const ICON_MAP = {
  error: '&#10060;',
  overload: '&#9889;',
  running: '&#9989;',
};

const COLOR_MAP = {
  error: '#f5222d',
  overload: '#faad14',
  running: '#52c41a',
};
const data =  {
    nodes: [
      { id: 'node-1', data: { location: 'East', status: 'error', ip: '192.168.1.2' }, style: { x: 100, y: 100 } },
      { id: 'node-2', data: { location: 'West', status: 'overload', ip: '192.168.1.3' }, style: { x: 300, y: 100 } },
      { id: 'node-3', data: { location: 'South', status: 'running', ip: '192.168.1.4' }, style: { x: 500, y: 100 } },
    ],
    edges: [
    {
      id: 'edge-1',
      source: 'node-1',
      target: 'node-2',
    },
    {
      id: 'edge-2',
      source: 'node-2',
      target: 'node-3',
    },
  ],
  }

let graph = null

function initGraph() {
   graph = new Graph({
  container: 'container',
  node: {
    type: 'html',
    style: {
      size: [120, 50],
      dx: 20,
      dy: 20,
      innerHTML: (d) => {
        const {
          data: { location, status, ip },
        } = d;
        const color = COLOR_MAP[status];
        return `
          <div 
          class="node2"
            style="
              width:100%; 
              height: 100%; 
              background: ${color}bb; 
              border: 1px solid ${color};
              border-radius: 5px;
              color: #fff;
              user-select: none;
              display: flex; 
              justify-content: center;
              align-items: center;
              padding: 10px;
              position: relative;
              "
          >
            <div class="delete-icon" data-node-id="${d.id}" style="
              position: absolute;
              top: -10px;
              right: -10px;
              width: 20px;
              height: 20px;
              background: #ff4d4f;
              border-radius: 50%;
              display: flex;
              justify-content: center;
              align-items: center;
              cursor: pointer;
              font-size: 12px;
              color: white;
            ">×</div>
            <div style="font-weight: bold;">
                ${location} Node
            </div>
          </div>`;
      },
    },
  },
  edge: {
    type: 'polyline',
    style: {
      endArrow: true,
      router: {
        type: 'orth',
      },
    },
    // 添加锚点配置
    sourceAnchor: {
      position: 'center',
      intersection: true,  // 允许与 combo 相交
    },
    targetAnchor: {
      position: 'center',
      intersection: true,  // 允许与 combo 相交
    },
  },
  combo: {
    type: 'rect',
    style: {
      radius: 8,
      padding: [10, 10],  // 增加内边距，让箭头有更好的连接空间
      fillOpacity: 0.1,
      stroke: '#91d5ff',
      fill: '#91d5ff',
    },
  },
    defaultCombo: {
      type: 'rect',
      style: {
        fillOpacity: 0.1,
        stroke: '#91d5ff',
      },
    },
  layout: {
    type: null,
  },
   // 启用历史记录
  enabledStack: true,
  behaviors: ['drag-element', 'drag-canvas', 'zoom-canvas'],
  plugins: [
    {
      type: 'history',
      key: 'history',
    },
  ],
});
// data.nodes = updateNodePositions(data.nodes);  // 添加这行
graph.setData(data);
graph.render();
}

function updateNodePositions(nodes) {
  return nodes.map(node => ({
    ...node,
    x: node.style.x,
    y: node.style.y,
    style: {
      ...node.style
    }
  }));
}

function handleContextMenuAction(action, nodeId) {
  const graphData = graph.getData();
  const currentNode = graphData.nodes.find(node => node.id === nodeId);
  const currentIndex = graphData.nodes.findIndex(node => node.id === nodeId);
  
  if (!currentNode) return;
  
  const newNodeId = `node-${graphData.nodes.length + 1}`;
  const spacing = 200;
  
  if (action === 'insertChild') {
    // 创建子节点
    const childNode = {
      id: newNodeId,
      data: {
        location: currentNode.data.location,
        status: currentNode.data.status,
        ip: '192.168.1.' + (graphData.nodes.length + 2)
      },
      style: {
        x: currentNode.style.x,
        y: currentNode.style.y - 100  // 在主节点上方50距离
      },
      combo: 'combo' + currentNode.id,  // 设置组合ID为父节点ID
      parentId: currentNode.id   // 设置父节点ID
    };
    console.log('childNode', childNode)
    // 添加子节点到数据中
    graphData.nodes.push(childNode);

    // 添加连接边
    graphData.edges.push({
      id: `edge-${Date.now()}`,
      source: currentNode.id,
      target: 'combo' + currentNode.id
    });
    console.log('graphData', graphData)
    // 如果还没有combos数组，创建一个
    if (!graphData.combos) {
      graphData.combos = [];
    }

    // 添加或更新combo
    const existingCombo = graphData.combos.find(combo => combo.id === currentNode.id);
    if (!existingCombo) {
      graphData.combos.push({
        id: 'combo' + currentNode.id,
        label: `${currentNode.data.location} Group`
      });
    }
    console.log('graphData2', graphData)
    // graph.clear();
    graph.setData(graphData);
    graph.render();
    return;
  }
  // 计算新节点的位置
  const nodeIndex = graphData.nodes.length;
  console.log(currentIndex)
  const row = Math.floor(nodeIndex / 5);
  const col = nodeIndex % 5;
  console.log(row, col)
  
  // 创建新节点，继承当前节点的属性
  const newNode = {
    id: newNodeId,
    data: {
      location: currentNode.data.location,
      status: currentNode.data.status,
      ip: '192.168.1.' + (graphData.nodes.length + 2)
    },
    style: {
      // x: currentNode.style.x,
      // y: currentNode.style.y
      x: 100 + col * 200,  // 每列间隔200像素
      y: 100  // 每行间隔150像素
    }
  };

  if (action === 'insertBefore') {
    // 将当前节点及其后续节点向右移动
    const nodesToMove = graphData.nodes.filter(node => 
      node.style.x >= currentNode.style.x
    );
    
    nodesToMove.forEach(node => {
      node.style.x += spacing;
    });
    
    // 处理连接关系
    const incomingEdge = graphData.edges.find(edge => edge.target === currentNode.id);
    if (incomingEdge) {
      incomingEdge.target = newNodeId;
      graphData.edges.push({
        id: `edge-${Date.now()}`,
        source: newNodeId,
        target: currentNode.id
      });
    }
    
  } else if (action === 'insertAfter') {
    // 设置新节点位置
    newNode.style.x = currentNode.style.x + spacing;
    
    // 将后续节点向右移动
    const nodesToMove = graphData.nodes.filter(node => 
      node.style.x > currentNode.style.x
    );
    
    nodesToMove.forEach(node => {
      node.style.x += spacing;
    });
    
    // 处理连接关系
    const outgoingEdge = graphData.edges.find(edge => edge.source === currentNode.id);
    if (outgoingEdge) {
      outgoingEdge.source = newNodeId;
      graphData.edges.push({
        id: `edge-${Date.now()}`,
        source: currentNode.id,
        target: newNodeId
      });
    } else {
      graphData.edges.push({
        id: `edge-${Date.now()}`,
        source: currentNode.id,
        target: newNodeId
      });
    }
  }
  
  // graphData.nodes.push(newNode);
  graphData.nodes.splice(currentIndex, 0, newNode);
  // 重新排列所有节点位置
  graphData.nodes = arrangeNodesPosition(graphData.nodes);
  graphData.edges = arrangeEdges(graphData.nodes);
  graphData.nodes = updateNodePositions(graphData.nodes);  // 添加这行

  // graph.clear();
  graph.setData(graphData);
  graph.render();
}

function addEventListenerClick() {
  const container = document.getElementById('container');
  const contextMenu = document.getElementById('contextMenu');
  let currentNodeId = null;

  // 右键菜单事件
  container.addEventListener('contextmenu', (e) => {
    e.preventDefault();
    const target = e.target.closest('.node2');
    if (target) {
      currentNodeId = target.querySelector('.delete-icon').getAttribute('data-node-id');
      contextMenu.style.display = 'block';
      contextMenu.style.left = `${e.pageX}px`;
      contextMenu.style.top = `${e.pageY}px`;
    }
  });

  // 点击其他地方隐藏菜单
  document.addEventListener('click', () => {
    contextMenu.style.display = 'none';
  });

  // 菜单项点击事件
  contextMenu.addEventListener('click', (e) => {
    const action = e.target.getAttribute('data-action');
    if (action && currentNodeId) {
      handleContextMenuAction(action, currentNodeId);
    }
  });
  document.getElementById('container').addEventListener('click', (e) => {
    if (e.target.classList.contains('delete-icon')) {
      const nodeId = e.target.getAttribute('data-node-id');
    // 获取当前图数据
    const graphData = graph.getData();
    // 过滤掉要删除的节点
    graphData.nodes = graphData.nodes.filter(node => node.id !== nodeId);
    // 过滤掉与该节点相关的边
    graphData.edges = graphData.edges.filter(edge => 
      edge.source !== nodeId && edge.target !== nodeId
    );
    // 更新图数据
    graph.setData(graphData);
    graph.render();
    }
  });
}

// 添加新的方法
const handleUndo = () => {
  const history = graph.getPluginInstance('history');
  if (history.canUndo()) history.undo();
};

const handleRedo = () => {
  const history = graph.getPluginInstance('history');
  console.log('history', history)
  if (history.canRedo()) history.redo();
};

const changeNodeColor = (status) => {
  const graphData = graph.getData();
  const lastNode = graphData.nodes[graphData.nodes.length - 1];
  const newNodeId = `node-${graphData.nodes.length + 1}`;
  
  // 计算新节点的位置
  const nodeIndex = graphData.nodes.length;
  const row = Math.floor(nodeIndex / 5);
  const col = nodeIndex % 5;
  
  // 创建新节点
  const newNode = {
    id: newNodeId,
    data: {
      location: status === 'error' ? 'East' : status === 'overload' ? 'West' : 'South',
      status: status,
      ip: '192.168.1.' + (graphData.nodes.length + 2)
    },
    style: {
      // x: (row%2 === 0) ? (100 + col * 200) : (900 - col * 200),  // 每列间隔200像素
      // y: 100 + row * 150   // 每行间隔150像素
      x: 100 + col * 200,  // 每列间隔200像素
      y: 100  // 每行间隔150像素
    }
  };

  // 创建新边
  const newEdge = {
    id: `edge-${graphData.edges.length + 1}`,
    source: lastNode.id,
    target: newNodeId
  };

  // 更新图数据
  graphData.nodes.push(newNode);
  graphData.edges.push(newEdge);
  
// 重新排列所有节点位置
 graphData.nodes = arrangeNodesPosition(graphData.nodes);
 graphData.nodes = updateNodePositions(graphData.nodes);  // 添加这行

  graph.setData(graphData);
  graph.render();
};

function arrangeNodesPosition(nodes) {
  nodes.forEach((node, index) => {
    const row = Math.floor(index / 5);  // 计算行数
    const col = index % 5;              // 计算列数
    
    if (row % 2 === 0) {
      // 奇数行：从左到右
      node.style.x = 100 + col * 200;
    } else {
      // 偶数行：从右到左
      node.style.x = 900 - col * 200;
    }
    node.style.y = 100 + row * 150;    // 每行间隔150像素
  });
  return nodes;
}

function arrangeEdges(nodes) {
  console.log('nodes2', nodes)
  let edges = []
  for(let i=0; nodes.length - 1 > i; i++) {
    edges.push({
      id: `edge-${i + 1}`,
      source: nodes[i].id,
      target: nodes[i+1].id,
    })
  }
  return edges;
}


function findInsertPosition(draggedNode, nodes) {
  const threshold = 50;
  const connectedNodes = [];

  // 找出所有通过边连接的节点对
  const graphData = graph.getData();
  graphData.edges.forEach(edge => {
    const sourceNode = nodes.find(node => node.id === edge.source);
    const targetNode = nodes.find(node => node.id === edge.target);
    if (sourceNode && targetNode) {
      connectedNodes.push({ sourceNode, targetNode });
    }
  });

  // 检查拖动的节点是否在任何连接的节点对之间
  for (const { sourceNode, targetNode } of connectedNodes) {
    if (sourceNode.id === draggedNode.id || targetNode.id === draggedNode.id) {
      continue;
    }

    const inBetweenX = (sourceNode.style.x < draggedNode.style.x && 
                       draggedNode.style.x < targetNode.style.x) ||
                      (targetNode.style.x < draggedNode.style.x && 
                       draggedNode.style.x < sourceNode.style.x);
                       
    const sameY = Math.abs(draggedNode.style.y - sourceNode.style.y) < threshold &&
                 Math.abs(draggedNode.style.y - targetNode.style.y) < threshold;

    if (inBetweenX && sameY) {
      return { sourceNode, targetNode };
    }
  }
  return null;
}

// 在 script setup 中添加指示器逻辑
function createInsertIndicator() {
  const indicator = document.createElement('div');
  indicator.className = 'insert-indicator';
  indicator.style.display = 'none';
  document.getElementById('container').appendChild(indicator);
  return indicator;
}

onMounted(() => {
  initGraph()
  addEventListenerClick()
  const insertIndicator = createInsertIndicator();
  console.log('insertIndicator', insertIndicator)

  let dragStartNode = null;
  let currentPosition = null;

  // 监听节点拖拽事件
  graph.on('node:dragstart', (evt) => {
    const nodeId = evt.target.id;
    const graphData = graph.getData();
    dragStartNode = graphData.nodes.find(node => node.id === nodeId);
    
    if (dragStartNode) {
      dragStartNode._originPosition = { 
        x: dragStartNode.style.x, 
        y: dragStartNode.style.y 
      };
    }
  });

  graph.on('node:drag', (evt) => {
    if (!dragStartNode) return;
    
    const graphData = graph.getData();
    const draggedNode = graphData.nodes.find(node => node.id === dragStartNode.id);
  
    // 检查是否可以插入到其他节点之间
    const insertPosition = findInsertPosition(draggedNode, graphData.nodes);
    currentPosition = insertPosition;
    
    // 显示或隐藏插入指示器
    if (currentPosition) {
      console.log(1)
      const { sourceNode, targetNode } = currentPosition;
      const x = targetNode.style.x - 20
      const y = sourceNode.style.y;
      
      insertIndicator.style.display = 'block';
      insertIndicator.style.left = `${x}px`;
      insertIndicator.style.top = `${y - 20}px`;
    } else {
      console.log(2)
      insertIndicator.style.display = 'none';
    }
  });

  graph.on('node:dragend', (evt) => {
  if (!dragStartNode) return;
  
  const graphData = graph.getData();
  const draggedNode = graphData.nodes.find(node => node.id === dragStartNode.id);
  
  if (currentPosition) {
    const { sourceNode, targetNode } = currentPosition;
    const spacing = 200; // 节点之间的间距
    
    // 删除原有边
    graphData.edges = graphData.edges.filter(edge => 
      !(edge.source === sourceNode.id && edge.target === targetNode.id)
    );
    
    // 添加新边
    graphData.edges.push(
      { id: `edge-${Date.now()}-1`, source: sourceNode.id, target: draggedNode.id },
      { id: `edge-${Date.now()}-2`, source: draggedNode.id, target: targetNode.id }
    );
    
    // 更新被拖动节点的位置到目标节点的位置
    draggedNode.style.x = targetNode.style.x;
    draggedNode.style.y = targetNode.style.y;
    
    // 将目标节点及其后续节点向右移动
    const nodesToMove = graphData.nodes.filter(node => 
      node.style.x >= targetNode.style.x && node.id !== draggedNode.id
    );
    
    nodesToMove.forEach(node => {
      node.style.x += spacing;
    });
  } else if (dragStartNode._originPosition) {
    // 恢复原始位置
    draggedNode.style.x = dragStartNode._originPosition.x;
    draggedNode.style.y = dragStartNode._originPosition.y;
  }
      
    // 隐藏插入指示器
    insertIndicator.style.display = 'none';
  // 重置状态
  dragStartNode = null;
  currentPosition = null;
    // 更新所有节点的位置
    graphData.nodes = updateNodePositions(graphData.nodes);
  console.log('graphData', graphData)
  // graph.clear();
  graph.setData(graphData);
  graph.render();
});
})
</script>

<template>
  <div class="workflow-container">
    <div class="button-group">
      <button class="btn red" @click="changeNodeColor('error')">红色节点</button>
      <button class="btn yellow" @click="changeNodeColor('overload')">黄色节点</button>
      <button class="btn green" @click="changeNodeColor('running')">绿色节点</button>
      <button class="btn undo" @click="handleUndo">撤销</button>
      <button class="btn redo" @click="handleRedo">重做</button>
    </div>
    <div id="container"></div>
    <div id="contextMenu">
      <div class="menu-item" data-action="insertBefore">节点前插入</div>
      <div class="menu-item" data-action="insertAfter">节点后插入</div>
      <div class="menu-item" data-action="insertChild">插入子节点</div>
      <div class="menu-item" data-action="insertBranch">插入分支节点</div>
    </div>
  </div>
</template>

<style lang="less" scoped>
#container {
      background-image: 
        linear-gradient(rgba(200, 200, 200, 0.3) 1px, transparent 1px),
        linear-gradient(90deg, rgba(200, 200, 200, 0.3) 1px, transparent 1px);
      background-size: 20px 20px;
      width: 1000px;
      height: 600px;
      margin: 20px auto;
    }
    
    #contextMenu {
      position: absolute;
      background: white;
      border: 1px solid #ccc;
      box-shadow: 2px 2px 5px rgba(0,0,0,0.2);
      display: none;
    }
    .menu-item {
      padding: 8px 15px;
      cursor: pointer;
    }
    .menu-item:hover {
      background: #f0f0f0;
    }
.workflow-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 20px;
}

.button-group {
  display: flex;
  gap: 10px;
  margin-top: 20px;
}

.btn {
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  color: white;
  font-weight: bold;
  transition: opacity 0.2s;
}

.btn:hover {
  opacity: 0.8;
}

.red {
  background-color: #f5222d;
}

.yellow {
  background-color: #faad14;
}

.green {
  background-color: #52c41a;
}

.undo, .redo {
  background-color: #1890ff;
}
#contextMenu {
  position: fixed;
  background: white;
  border: 1px solid #ccc;
  box-shadow: 2px 2px 5px rgba(0,0,0,0.2);
  display: none;
  z-index: 1000;
}

.menu-item {
  padding: 8px 15px;
  cursor: pointer;
  white-space: nowrap;
}

.menu-item:hover {
  background: #f0f0f0;
}
</style>
<style>
.insert-indicator {
  position: absolute;
  width: 120px;
  height: 40px;
  pointer-events: none;
  z-index: 1000;
  display: flex;
  justify-content: center;
  align-items: center;
}

.insert-indicator::before {
  content: '';
  position: absolute;
  width: 100%;
  height: 2px;
  background: linear-gradient(90deg, 
    transparent 0%, 
    #1890ff 20%, 
    #1890ff 80%, 
    transparent 100%
  );
  animation: expandLine 1.5s ease-in-out infinite;
}

.insert-indicator::after {
  content: '拖拽至此';
  position: absolute;
  color: #1890ff;
  font-size: 12px;
  background: rgba(255, 255, 255, 0.9);
  padding: 2px 8px;
  border-radius: 10px;
  white-space: nowrap;
  box-shadow: 0 2px 4px rgba(24, 144, 255, 0.2);
  animation: fadeInOut 1.5s ease-in-out infinite;
}

@keyframes expandLine {
  0%, 100% {
    transform: scaleX(0.7);
    opacity: 0.5;
  }
  50% {
    transform: scaleX(1);
    opacity: 1;
  }
}

@keyframes fadeInOut {
  0%, 100% {
    opacity: 0.7;
    transform: translateY(-15px);
  }
  50% {
    opacity: 1;
    transform: translateY(-18px);
  }
}
</style>
