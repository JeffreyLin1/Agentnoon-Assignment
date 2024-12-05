<script setup>
        import { ref, watchEffect } from 'vue';
        import * as d3 from 'd3';


        const props = defineProps({
        data: Object
        });

        const svgContainer = ref(null);


    
    const showModal = ref(false);
    const selectedNode = ref(null);
    const colorScale = d3.scaleOrdinal(['#CB9DF0', '#F0C1E1', '#D4F6FF', '#FFF6E3','#C9E9D2','#FFB38E','#BC7C7C','#FFECC8','#A5B68D','#CDC1FF','#F1F3C2','#F8EDE3','#606676', '#DFD3C3','#F1F8E8', '#DBB5B5','#F7DCB9','#FFEFEF','#BED7DC','#FFBE98']);
    function openModal(node) {
        const fillColor = colorScale(node.position); 
        const strokeColor = d3.color(fillColor)?.darker(1.1).formatHex(); 
        selectedNode.value = {
            ...node,
            fillColor,
            strokeColor,
        }; 
        showModal.value = true; 
    }

    function closeModal() {
        showModal.value = false;
    }
    function calculateTotalDescendants(node, memo = new Map()) {

    if (memo.has(node)) {
        return memo.get(node);
    }

    if (!node.children) {
        node.data.totalDescendants = 0;
        memo.set(node, 0);
        return 0;
    }

    const totalDescendants = node.children.reduce((sum, child) => {
        return sum + calculateTotalDescendants(child, memo) + 1; 
    }, 0);

    node.data.totalDescendants = totalDescendants;
    memo.set(node, totalDescendants); 

    return totalDescendants;
}

function calculateCosts(node, memo = new Map()) {
    if (memo.has(node)) {
        return memo.get(node);
    }

    if (!node.children) {
        const salary = parseFloat(node.data.data.Salary) || 0;
        const costs = {
            managementCost: 0,
            icCost: salary,
            totalCost: salary,
            managementCostRatio: 0,
        };
        node.data.managementCost = costs.managementCost;
        node.data.icCost = costs.icCost;
        node.data.totalCost = costs.totalCost;
        node.data.managementCostRatio = costs.managementCostRatio;

        memo.set(node, costs); // Cache the result
        return costs;
    }

    let managementCost = 0;
    let icCost = 0;
    let totalCost = parseFloat(node.data.data.Salary) || 0;


    node.children.forEach((child) => {
        const childCosts = calculateCosts(child, memo);
        managementCost += childCosts.managementCost;
        icCost += childCosts.icCost;
        totalCost += childCosts.totalCost;
    });

    const salary = parseFloat(node.data.data.Salary) || 0;
    if (node.children.length > 0) {
        managementCost += salary; 
    }

    const costs = {
        managementCost,
        icCost,
        totalCost,
        managementCostRatio: icCost > 0 ? (icCost / managementCost).toFixed(2) : 0,
    };


    node.data.managementCost = costs.managementCost;
    node.data.icCost = costs.icCost;
    node.data.totalCost = costs.totalCost;
    node.data.managementCostRatio = costs.managementCostRatio;

    memo.set(node, costs); 

    return costs;
}

    function formatNumber(num) {
        if (num >= 1e6) {
            return (num / 1e6).toFixed(0) + ' M';
        } else if (num >= 1e3) {
            return (num / 1e3).toFixed(0) + ' k'; 
        } else {
            return num.toFixed(3); 
        }
    }


    function customLink(d) {
        const sourceY = d.source.y+150; 
        const targetX = d.target.x;
        const targetY = d.target.y;


        return `
            M${d.source.x},${d.source.y}
            V${sourceY}
            H${targetX}
            V${targetY}
        `;
    }

        watchEffect(() => {
        if (!props.data) return;


        if (svgContainer.value) {
            svgContainer.value.innerHTML = '';
        }


        const marginTop = 50;
        const marginRight = 300;
        const marginBottom = 50;
        const marginLeft = 300;


        const dx = 120;
        const dy = 200;

        const tree = d3.tree().nodeSize([dx, dy]);
        const root = d3.hierarchy(props.data);
        const uniqueDepartments = Array.from(new Set(root.descendants().map((d) => d.data.position)));
        colorScale.domain(uniqueDepartments);
        root.x0 = dy/2;
        root.y0 = dx/2;
        root.descendants().forEach((d) => {
        d.x += d.depth * 200;
        });
        calculateTotalDescendants(root, new Map());
        calculateCosts(root, new Map());
        root.descendants().forEach((d, i) => {
        d.id = i;
        d._children = d.children;


        d.data.name = d.data.data.Name;
        d.data.position = d.data.data.Department
        d.data.title = d.data.data["Job Title"];
        d.data.location = d.data.data.Location;
        d.data.level = d.data.data.level;
        d.data.performance = d.data.data.Performance;
        d.data.project = d.data.data.Project;
        d.data.entity = d.data.data.Entity;
        d.data.salary = d.data.data.Salary;
        d.data.bonus = d.data.data.Bonus;
        d.data.email = d.data.data.Email;
        d.data.status = d.data.data.Status;



        if (d.depth) d.children = null;
        });
        const svg = d3.select(svgContainer.value)
        .attr('width', '100%')
        .attr('height', '100%')
        .attr('preserveAspectRatio', 'xMidYMid meet')
        .attr('style', 'max-width: 100%; max-height: "100%"; font: 10px sans-serif; user-select: none;');
        const zoomGroup = svg.append('g');
        const defs = svg.append('defs');

    defs.append('filter')
        .attr('id', 'dropShadow') 
        .attr('x', '-20%')
        .attr('y', '-20%')
        .attr('width', '140%') 
        .attr('height', '140%')
        .append('feDropShadow')
        .attr('dx', 2) 
        .attr('dy', 2) 
        .attr('stdDeviation', 2) 
        .attr('flood-color', 'rgba(0, 0, 0, 0.5)'); 

    const gLink = zoomGroup.append('g')
        .attr('fill', 'none')
        .attr('stroke', '#555')
        .attr('stroke-opacity', 0.4)
        .attr('stroke-width', 1.5);

    const gNode = zoomGroup.append('g')
        .attr('cursor', 'pointer')
        .attr('pointer-events', 'all');

        function update(event, source) {
        const nodes = root.descendants().reverse();
        const links = root.links();

        tree(root);

        let left = root;
        let right = root;
        let top = root;
        let bottom = root;

        root.eachBefore(node => {
            if (node.x < left.x) left = node;
            if (node.x > right.x) right = node;
            if (node.y < top.y) top = node;
            if (node.y > bottom.y) bottom = node;
        });

        const width = Math.max(1000, right.x - left.x + marginLeft + marginRight);
        const height = Math.max(800, bottom.y + marginTop + marginBottom); 
        const transition = svg.transition()
            .duration(400)
            .attr('viewBox', [
                left.x - marginLeft, 
                top.y - marginTop, 
                width,
                height*2
            ])
            .tween('resize', window.ResizeObserver ? null : () => () => svg.dispatch('toggle'));

        const node = gNode.selectAll('g').data(nodes, d => d.id);

        const nodeEnter = node.enter().append('g')
            .attr('transform', d => `translate(${source.y0},${source.x0})`)
            .attr('fill-opacity', 0)
            .attr('stroke-opacity', 0)
            .on('click', (event, d) => {
                d.children = d.children ? null : d._children;
                update(event, d);
            });

        nodeEnter.append('rect')
            .attr('width', 100)
            .attr('height', 138)
            .attr('x', -45)
            .attr('y', -15)
            .attr('fill', (d) => colorScale(d.data.position))
            .attr('stroke', (d) => {
            const fillColor = d3.color(colorScale(d.data.position)); 
            if (fillColor) {
                const darkerColor = fillColor.darker(1.1); 
                return darkerColor.formatHex(); 
            }
            return '#939393'; 
        })
            .attr('stroke-width', 1.5)
            .attr('rx', 4)
            .attr('ry', 4)
            .attr('filter', 'url(#dropShadow)');
        nodeEnter.append('rect')
            .attr('width', 90) 
            .attr('height', 105) 
            .attr('x', -40) 
            .attr('y', 14) 
            .attr('fill', '#fff') 
            .attr('stroke', (d) => {
            const fillColor = d3.color(colorScale(d.data.position)); 
            if (fillColor) {
                const darkerColor = fillColor.darker(1.1); 
                return darkerColor.formatHex(); 
            }
            return '#939393'; 
        }) 
            .attr('stroke', (d) => {
            const fillColor = d3.color(colorScale(d.data.position)); 
            if (fillColor) {
                const darkerColor = fillColor.darker(1.1); 
                return darkerColor.formatHex(); 
            }
            return '#939393'; 
        }) 
            .attr('stroke-width', 1.2)
            .attr('rx', 3)
            .attr('ry', 3);

        nodeEnter.append('circle')
            .attr('cx', 3) 
            .attr('cy', -19) 
            .attr('r', 16) 
            .attr('fill', '#fff') 
            .attr('stroke', '#939393') 
            .attr('stroke-width', 1.5);
        
        nodeEnter.append('text')
            .attr('dx', 3)
            .attr('dy', -15) 
            .attr('text-anchor', 'middle') 
            .attr('font-size', '10px') 
            .attr('font-weight', 'bold') 
            .text((d) => {
            
            const name = d.data.name;
            const initials = name.split(' ').map(word => word[0]).join('').toUpperCase();
            return initials;
        })
        .attr('fill', '#939393'); 

        nodeEnter.append('text')
            .attr('dx', '0.35em')
            .attr('text-anchor', 'middle')
            .attr('y', 8)
            .attr('font-size', '8px')
            .attr('font-weight', 'bold')
            .text(d => d.data.name)
            .attr('fill', '#000');
        nodeEnter.append('text')
            .attr('dx', '0.7em')
            .attr('text-anchor', 'middle')
            .attr('y', 40)
            .attr('font-size', '7px')
            .attr('font-weight', 'bold')
            .attr('fill', '#000')
            .each(function (d) {
            const textElement = d3.select(this);
            const words = d.data.position.split(' ');
            let line = '';
            let lineNumber = 0;
            const lineHeight = 8; 
            const maxLineLength = 24; 

            words.forEach((word, index) => {
                if ((line + word).length > maxLineLength) {
                    textElement.append('tspan')
                        .attr('x', 0)
                        .attr('dy', lineNumber === 0 ? 0 : lineHeight)
                        .text(line.trim());
                    line = '';
                    lineNumber++;
                }
                line += `${word} `;
            });

            
            textElement.append('tspan')
                .attr('x', 0)
                .attr('dy', lineNumber === 0 ? 0 : lineHeight)
                .text(line.trim());
        });

        nodeEnter.append('text')
            .attr('dx', '0.35em')
            .attr('text-anchor', 'middle')
            .attr('y', 24)
            .attr('font-size', '7px')
            .attr('font-weight', 'bold')
            .attr('fill', '#939393')
            .each(function (d) {
            const textElement = d3.select(this);
            const words = d.data.title.split(' '); 
            let line = '';
            let lineNumber = 0;
            const lineHeight = 8; 
            const maxLineLength = 24; 

            words.forEach((word, index) => {
                if ((line + word).length > maxLineLength) {
                    textElement.append('tspan')
                        .attr('x', 0)
                        .attr('dy', lineNumber === 0 ? 0 : lineHeight)
                        .text(line.trim());
                    line = '';
                    lineNumber++;
                }
                line += `${word} `;
            });

         
            textElement.append('tspan')
                .attr('x', 0)
                .attr('dy', lineNumber === 0 ? 0 : lineHeight)
                .text(line.trim());
        });



        nodeEnter.append('text')
            .attr('dx', '0.7em')
            .attr('text-anchor', 'middle')
            .attr('y', 60)
            .attr('font-size', '6px')
            .attr('font-weight', 'bold')
            .text(d => d.data.location)
            .attr('fill', '#000');
            
        nodeEnter.append('text')
            .attr('dx', '0.35em')
            .attr('text-anchor', 'middle')
            .attr('y', 80)
            .attr('font-size', '6px')
            .attr('font-weight', 'bold')
            .text(d => d.data.performance)
            .attr('fill', '#000');

        nodeEnter.append('text')
            .attr('dx', '0.35em')
            .attr('text-anchor', 'middle')
            .attr('y', 100)
            .attr('font-size', '6px')
            .attr('font-weight', 'bold')
            .text((d) => `Total Descendants: ${d.data.totalDescendants}`)
            .attr('fill', '#000');


        nodeEnter.append('text')
            .attr('dx', '3em')
            .attr('text-anchor', 'right')
            .attr('y', 114)
            .attr('font-size', '6px')
            .attr('font-weight', 'bold')
            .attr('fill', '#939393')
            .style('text-decoration', 'underline') 
        .text('Details >>')
        .style('cursor', 'pointer')
        .on('mouseover', function (event, d) {
            d3.select(this)
                .attr('fill', '#bfbfbf') 
                .style('text-decoration', 'underline'); 
        })
        .on('mouseout', function (event, d) {
            d3.select(this)
                .attr('fill', '#939393') 
                .style('text-decoration', 'underline'); 
        })
        .on('click', (event, d) => {
            event.stopPropagation(); 
            openModal(d.data); 
        });
        node.merge(nodeEnter).transition(transition)
            .attr('transform', d => `translate(${d.x},${d.y})`)
            .attr('fill-opacity', 1)
            .attr('stroke-opacity', 1);

        node.exit().transition(transition).remove()
            .attr('transform', d => `translate(${source.y},${source.x})`)
            .attr('fill-opacity', 0)
            .attr('stroke-opacity', 0);


        const link = gLink.selectAll('path').data(links, (d) => d.target.id);

    const linkEnter = link.enter().append('path')
        .attr('d', (d) => {
            const o = { x: source?.x0, y: source?.y0};
            return `M${o.x},${o.y} V${o.y} H${o.x} V${o.y}`;
        });

    link.merge(linkEnter)
        .transition()
        .duration(400)
        .attr('d', customLink);

        link.exit()
        .transition()
        .duration(400)
        .attr('d', (d) => {
            
            const o = { x: source?.x0 , y: source?.y0 };
            return `M${o.x},${o.y} V${o.y} H${o.x} V${o.y}`;
        })
        .remove(); 


        root.eachBefore(d => {
            d.x0 = d.x;
            d.y0 = d.y;
        });

    }
        update(null, root);
    });
</script>

<template>
    <div>
      <!-- SVG Container -->
      <svg ref="svgContainer" class="w-full h-full"></svg>
  
      <!-- Modal -->
      <div
  v-if="showModal"
  class="fixed top-0 right-0 w-72 h-full bg-white border-l-4 border-gray-300 shadow-lg p-6 overflow-y-auto z-50 modal-container"
  :style="{ boxShadow: `-4px 0 6px ${selectedNode?.strokeColor || 'rgba(0, 0, 0, 0.1)'}` }"
>
  <!-- Icon with Initials -->
  <svg
    class="absolute -top-12 left-1/2 transform -translate-x-1/2"
    width="150"
    height="300"
  >
    <circle
      cx="80"
      cy="150"
      r="60"
      :fill="'#fff'"
      :stroke="selectedNode?.strokeColor"
      stroke-width="2"
    />
    <text
      x="80"
      y="155"
      text-anchor="middle"
      font-size="40"
      font-weight="bold"
      :fill="selectedNode?.strokeColor"
      dominant-baseline="middle"
    >
      {{ selectedNode?.name.split(' ').map(word => word[0]).join('').toUpperCase() }}
    </text>
  </svg>

  <!-- Content Section -->
  <div class="mt-[150px]"> <!-- Add margin-top to create space -->
    <h1
      class="text-3xl font-bold text-gray-800 mb-4 "
      :style="{ color: selectedNode?.strokeColor }"
    >
      {{ selectedNode?.name }}
    </h1>
    <p><strong>Email:</strong> {{ selectedNode?.email }}</p>
    <p><strong>Department:</strong> {{ selectedNode?.position }}</p>
    <p><strong>Job Title:</strong> {{ selectedNode?.title }}</p>
    <p><strong>Location:</strong> {{ selectedNode?.location }}</p>
    <p><strong>Entity:</strong> {{ selectedNode?.entity }}</p>
    <p><strong>Performance:</strong> {{ selectedNode?.performance }}</p>
    <p><strong>Project:</strong> {{ selectedNode?.project }}</p>
    <p><strong>Salary:</strong> ${{ formatNumber(selectedNode?.salary) }}</p>
    <p><strong>Bonus:</strong> ${{ formatNumber(selectedNode?.bonus) }}</p>
    <p><strong>Status:</strong> {{ selectedNode?.status }}</p>
    <p><strong>Total Cost:</strong> ${{ formatNumber(selectedNode?.totalCost) }}</p>
    <p><strong>Management Cost:</strong> ${{ formatNumber(selectedNode?.managementCost) }}</p>
    <p><strong>IC Cost:</strong> ${{ formatNumber(selectedNode?.icCost) }}</p>
    <p><strong>Total Descendants:</strong> {{ selectedNode?.totalDescendants }}</p>
    <button
      @click="closeModal"
      class="mt-6 px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700 transition w-full"
    >
      Close
    </button>
  </div>
</div>

    </div>
  </template>
  <style scoped>
  .modal-container {
      animation-name: animateside;
      animation-duration: 0.4s;
  }

  @keyframes animateside {
      from {
          right: -300px;
          opacity: 0;
      }
      to {
          right: 0;
          opacity: 1;
      }
  }
</style>