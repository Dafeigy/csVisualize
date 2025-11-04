<template>
  <div id="container" class="max-w-[80%] h-[95%] aspect-video flex justify-center items-center bg-[#202124] flex-row  text-white rounded-l-2xl border border-[#323233]">
    <div id="sidebar" class="w-1/6 h-full">
      <div id="fileload" class="flex flex-col justify-center items-center bg-[#202124] p-2 h-[15%] border-l border-[#323233] rounded-tl-2xl">
        <!-- <button @click="triggerFileUpload" class="w-[80%] text-white bg-[#02DA7F] h-[40%] cursor-pointer rounded-md text-sm hover:bg-[#02b875] duration-200 mb-2">
          Load File
        </button> -->
        <input ref="fileInput" type="file" accept=".csv" @change="handleFileUpload" class="hidden">
        <div class="w-[80%] mt-2 flex rounded-md overflow-hidden">
          <button 
            :class="['flex-1 py-1 text-sm transition-all duration-200', encoding === 'utf-8' ? 'bg-[#02DA7F] text-white' : 'bg-[#2a2b2d] text-gray-400 hover:bg-[#3a3b3d]']"
            @click="encoding = 'utf-8'"
          >
            UTF-8
          </button>
          <button 
            :class="['flex-1 py-1 text-sm transition-all duration-200', encoding === 'gbk' ? 'bg-[#02DA7F] text-white' : 'bg-[#2a2b2d] text-gray-400 hover:bg-[#3a3b3d]']"
            @click="encoding = 'gbk'"
          >
            GBK
          </button>
        </div>
        <div v-if="loading" class="text-xs text-[#02DA7F] mt-1">加载中...</div>
      </div>
      <div id="controls" class="justify-between h-[85%] bg-[#1A1B1D] rounded-bl-2xl border-l border-[#323233] overflow-hidden flex flex-col">
        <div id="factors" class="p-2 border-b border-[#323233] h-[80%] flex flex-col text-gray-400">  
          <div class="text-sm font-semibold text-[#02DA7F] h-[2%]">Factors</div>
          <div class="overflow-y-scroll h-[98%] mt-4">
            <div v-for="header in headers" :key="header" class="text-sm mb-1 flex items-center">
              <input type="checkbox" :value="header" v-model="selectedMetrics" class="mr-2 accent-[#02DA7F]">
              <span>{{ header }}</span>
            </div>
            <div v-if="headers.length === 0" class="text-sm text-gray-500 text-center py-2">Please Load File first.</div>
          </div>
        </div>
        <div id="timeselect" class="p-2 flex-1 flex flex-col h-[20%]">
          <div class="text-sm font-semibold text-[#02DA7F]">Time Range</div>
          <div v-if="csvData.length > 0" class="flex-1 flex flex-col">
            <div class="grid grid-cols-2 gap-2 mb-3">
              <div class="flex flex-col">
                <label class="text-xs text-gray-400 mb-1">Start:</label>
                <div class="flex items-center">
                  <input type="number" v-model.number="timeRange.start" :min="0" :max="timeRange.end" 
                         class="bg-[#2a2b2d] text-white text-xs p-1 rounded border border-[#555] w-full" 
                         @change="handleStartInputChange">
                </div>
              </div>
              <div class="flex flex-col">
                <label class="text-xs text-gray-400 mb-1">End:</label>
                <div class="flex items-center">
                  <input type="number" v-model.number="timeRange.end" :min="timeRange.start" :max="csvData.length - 1" 
                         class="bg-[#2a2b2d] text-white text-xs p-1 rounded border border-[#555] w-full" 
                         @change="handleEndInputChange">
                </div>
              </div>
            </div>
            <div class="text-xs text-gray-500 mb-2">范围: 0 - {{ csvData.length - 1 }}</div>
            <input type="range" v-model.number="timeRange.start" :min="0" :max="csvData.length - 1" 
                   class="appearance-none mb-3" @input="updateChart">
            <input type="range" v-model.number="timeRange.end" :min="0" :max="csvData.length - 1" 
                   class="appearance-none " @input="updateChart">
          </div>
          <div v-else class="text-sm text-gray-500 text-center py-2">Please Load File first.</div>
        </div>
      </div>
    </div>
    <div id="chartsection" class="bg-[#1A1B1D] w-5/6 h-full border-l border-[#323233] flex flex-col justify-between items-center">
      <div id="fileinfo" class="h-[3%] w-[99%] bg-[#202124] text-xs flex items-center justify-start my-[1%] text-gray-500 rounded">
        <div v-if="currentFile" class="cursor-pointer flex items-center justify-center text-gray-400 truncate bg-[#333] h-[90%] text-xs rounded px-1 mx-1 hover:text-gray-100 transition-colors duration-200">
          {{ currentFile.name }}
          <div class="flex p-0 items-center justify-center text-center ml-2 text-white bg-[#333] rounded-full aspect-square cursor-pointer h-[70%] text-xs hover:bg-[#d15858] duration-200">
            ×
          </div>
        </div>
        <div v-else class="text-[#02DA7F] whihespace-nowrap bg-[#333] h-[90%] text-xs rounded mx-1 "></div>
        <div id="addfile" @click="triggerFileUpload" class="flex items-center justify-center text-center mx-2 text-white bg-[#333] rounded aspect-square cursor-pointer h-[80%] text-xs hover:bg-[#02b875] duration-200  ">
          +
        </div>
      </div>
      <div class="h-[80%] w-[99%] bg-[#202124] rounded-2xl justify-center flex items-center">
        <div id="chart" class="w-[98%] h-[98%]"></div>
      </div>
      <div id="bash" class="h-[16%] w-[99%] text-xs flex text-gray-500 my-[1%] flex-col">
        <div class="w-full flex items-center flex-col h-full"> 
          <div id="factor-select" class="w-full h-4/9 flex p-2 justify-start ">
            <div class="w-full flex gap-2  overflow-y-hidden overflow-x-auto">
              <button 
                @click="selectedFactor = ''"
                :class="['px-3 py-1 text-xs rounded-md transition-all duration-200 text-nowrap cursor-pointer', 
                         selectedFactor === '' ? 'bg-[#2a2b2d] text-gray-400' : 'bg-[#2a2b2d] text-gray-400 hover:bg-[#3a3b3d]']"
              >
                Reset
              </button>
              <button 
                v-for="header in headers" 
                :key="header"
                @click="selectedFactor = header"
                :class="['px-3 py-1 text-xs rounded-md transition-all duration-200 text-nowrap cursor-pointer', 
                         selectedFactor === header ? 'bg-[#02DA7F] text-white' : 'bg-[#2a2b2d] text-gray-400 hover:bg-[#3a3b3d]']"
              >
                {{ header }}
              </button>
            </div>
          </div>
          <div id="factor-summary" class="flex w-full h-5/9 items-center">
            <div v-if="selectedFactor" class="flex mx-1 justify-start">
              <div class="mr-4">
                <div class="font-semibold text-[#02DA7F] mb-1 mx-2">全局数据</div>
                <div class="flex mx-1">
                  <div class="mx-2">均值:</div>
                  <div>{{ factorStats.mean || '-' }}</div>
                  <div class="mx-2">最小值:</div>
                  <div>{{ factorStats.min || '-' }}</div>
                </div>
                <div class="flex mx-1">
                  <div class="mx-2">最大值:</div>
                  <div>{{ factorStats.max || '-' }}</div>
                  <div class="mx-2">方差:</div>
                  <div>{{ factorStats.variance || '-' }}</div>
                </div>
              </div>
              <div>
                <div class="font-semibold text-[#02DA7F] mb-1 mx-2">局部数据({{ timeRange.start }} - {{ timeRange.end }})</div>
                <div class="flex mx-1">
                  <div class="mx-2">均值:</div>
                  <div>{{ timeRangeStats.mean || '-' }}</div>
                  <div class="mx-2">最小值:</div>
                  <div>{{ timeRangeStats.min || '-' }}</div>
                </div>
                <div class="flex mx-1">
                  <div class="mx-2">最大值:</div>
                  <div>{{ timeRangeStats.max || '-' }}</div>
                  <div class="mx-2">方差:</div>
                  <div>{{ timeRangeStats.variance || '-' }}</div>
                </div>
              </div>
            </div>
            <div v-else class="text-gray-500 text-xs flex justify-center items-center"></div>
          </div>

        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, watch } from 'vue';
import * as echarts from 'echarts';

// 状态管理
const fileInput = ref(null);
const csvData = ref([]);
const headers = ref([]);
const selectedMetrics = ref([]);
const chartInstance = ref(null);
const timeRange = ref({ start: 0, end: 99 });
const loading = ref(false);
const encoding = ref('utf-8');
const currentFile = ref(null); // 保存当前上传的文件引用
const selectedFactor = ref(''); // 当前选择的factor
const factorStats = ref({ mean: null, min: null, max: null, variance: null }); // factor统计信息
const timeRangeStats = ref({ mean: null, min: null, max: null, variance: null }); // 基于TimeRange的factor统计信息
const activeDataLength = ref(0);

// 触发文件上传
const triggerFileUpload = () => {
  fileInput.value?.click();
};

// 处理文件上传
const handleFileUpload = (event) => {
  const file = event.target.files[0];
  if (!file) return;
  
  // 保存当前文件引用
  currentFile.value = file;
  
  // 加载文件
  loadCSVFile(file, encoding.value);
  
  // 清空input以允许重新选择同一文件
  event.target.value = '';
};

// 加载CSV文件的函数
const loadCSVFile = (file, fileEncoding) => {
  if (!file) return;
  
  loading.value = true;
  
  // 使用FileReader读取文件
  const reader = new FileReader();
  
  reader.onload = (e) => {
    try {
      let content = e.target.result;
      
      // 解析CSV内容
      const lines = content.split('\n').filter(line => line.trim());
      
      if (lines.length === 0) {
        throw new Error('文件内容为空');
      }
      
      // 解析表头
      headers.value = lines[0].split(',').map(h => h.trim());
      
      // 解析数据
      csvData.value = [];
      for (let i = 1; i < lines.length; i++) {
        const values = lines[i].split(',');
        const row = {};
        headers.value.forEach((header, index) => {
          // 尝试转换为数字
          const numValue = parseFloat(values[index]);
          row[header] = isNaN(numValue) ? values[index]?.trim() || '' : numValue;
        });
        csvData.value.push(row);
      }
      
      // 重置选中的指标、时间范围和factor选择
    selectedMetrics.value = [];
    selectedFactor.value = '';
    timeRange.value = { 
      start: 0, 
      end: Math.min(100, csvData.value.length - 1) 
    };
      
      console.log('CSV数据已加载:', csvData.value.length, '行，编码:', fileEncoding);
    } catch (error) {
      console.error('解析CSV文件失败:', error);
    } finally {
      loading.value = false;
    }
  };
  
  reader.onerror = () => {
    console.error('文件读取失败');
    loading.value = false;
  };
  
  reader.readAsText(file, fileEncoding);
};

// 初始化图表
const initChart = () => {
  if (!chartInstance.value) {
    chartInstance.value = echarts.init(document.getElementById('chart'));
    
    // 设置响应式
    window.addEventListener('resize', () => {
      chartInstance.value?.resize();
    });
  }
};

// 处理Start输入变化
const handleStartInputChange = () => {
  // 确保start值有效
  let start = parseInt(timeRange.value.start);
  if (isNaN(start) || start < 0) {
    start = 0;
  } else if (start > timeRange.value.end) {
    start = timeRange.value.end;
  } else if (start >= csvData.value.length) {
    start = csvData.value.length - 1;
  }
  timeRange.value.start = start;
  updateChart();
};

// 处理End输入变化
const handleEndInputChange = () => {
  // 确保end值有效
  let end = parseInt(timeRange.value.end);
  if (isNaN(end) || end >= csvData.value.length) {
    end = csvData.value.length - 1;
  } else if (end < timeRange.value.start) {
    end = timeRange.value.start;
  } else if (end < 0) {
    end = 0;
  }
  timeRange.value.end = end;
  updateChart();
};

// 更新图表
const updateChart = () => {
  if (!chartInstance.value || csvData.value.length === 0) {
    return;
  }
  
  // 当没有选中任何指标时，清除图表内容
  if (selectedMetrics.value.length === 0) {
    chartInstance.value.clear();
    return;
  }
  
  // 确保时间范围有效
  const start = Math.max(0, Math.min(timeRange.value.start, csvData.value.length - 1));
  const end = Math.max(start, Math.min(timeRange.value.end, csvData.value.length - 1));
  
  // 只有当值发生变化时才更新，避免无限循环
  if (timeRange.value.start !== start) {
    timeRange.value.start = start;
  }
  if (timeRange.value.end !== end) {
    timeRange.value.end = end;
  }
  
  // 准备数据
  const filteredData = csvData.value.slice(start, end + 1);
  // 假设第一列是时间列
  const timeAxis = filteredData.map(row => row[headers.value[0]]);
  
  // 准备series数据
  const series = selectedMetrics.value.map(metric => {
    return {
      name: metric,
      type: 'line',
      data: filteredData.map(row => row[metric]),
      smooth: true
    };
  });
  
  // 根据数据点数量确定tooltip配置
  activeDataLength.value = filteredData.length
  const dataPointCount = activeDataLength.value;
  const tooltipConfig = {
    trigger: 'item',
    backgroundColor: 'rgba(255, 255, 255, 0.9)',
    borderColor: '#ccc',
    formmater: function(params) {
      console.log("current params", params)
      const data = params[0];
      return `
        <div style="font-weight: bold; margin-bottom: 2px;">${data.seriesName}</div>
        <div style="display: flex; align-items: center;">
          <span style="display: inline-block; width: 8px; height: 8px; background-color: ${data.color}; border-radius: 50%; margin-right: 6px;"></span>
          <span>${data.axisValue} ${data.value}</span>
        </div>
      `;
    },
    textStyle: {
      color: '#333'
    },
    axisPointer: {
      type: 'cross',
      label: {
        backgroundColor: '#555'
      }
    }
  };
  
  
  // 设置图表配置
  const option = {
    tooltip: tooltipConfig,
    legend: {
      data: selectedMetrics.value,
      textStyle: {
        color: '#fff'
      },
      top: 30
    },
    grid: {
      left: '3%',
      right: '4%',
      bottom: '3%',
      containLabel: true
    },
    xAxis: {
      axisLine: {
show: false // 隐藏X轴轴线
},
      type: 'category',
      boundaryGap: false,
      data: timeAxis,
      axisLabel: {
        color: '#ccc'
      },
      axisLine: {
        lineStyle: {
          color: '#555'
        }
      }
    },
    yAxis: {
      type: 'value',
      axisLabel: {
        color: '#ccc'
      },
      axisLine: {
        lineStyle: {
          color: '#555'
        }
      },
      splitLine: {
        lineStyle: {
          color: '#333'
        }
      }
    },
    series: series,
    // 添加timeline组件
  };
  
  // 使用notMerge:true确保图表完全重新渲染，不合并旧配置
  chartInstance.value.setOption(option, true);
};

// 监听选中指标变化
watch(selectedMetrics, () => {
  updateChart();
}, { deep: true });

// 监听factor选择变化，计算统计信息
watch(selectedFactor, (newFactor) => {
  if (newFactor && csvData.value.length > 0) {
    calculateFactorStats(newFactor);
  } else {
    // 重置统计信息
    factorStats.value = { mean: null, min: null, max: null, variance: null };
    timeRangeStats.value = { mean: null, min: null, max: null, variance: null };
  }
});

// 监听timeRange变化，更新TimeRange统计信息
watch(timeRange, () => {
  if (selectedFactor.value && csvData.value.length > 0) {
    calculateTimeRangeStats(selectedFactor.value);
  }
}, { deep: true });

// 监听编码变化，当有文件且编码变化时重新加载
watch(encoding, (newEncoding, oldEncoding) => {
  if (currentFile.value && newEncoding !== oldEncoding) {
    console.log(`编码从 ${oldEncoding} 切换到 ${newEncoding}，重新加载文件`);
    loadCSVFile(currentFile.value, newEncoding);
  }
});

// 计算factor的统计信息（忽略空值）
const calculateFactorStats = (factor) => {
  // 过滤出有效的数值数据
  const validValues = csvData.value
    .map(row => row[factor])
    .filter(value => value !== null && value !== undefined && value !== '' && typeof value === 'number' && !isNaN(value));
  
  if (validValues.length === 0) {
    factorStats.value = { mean: null, min: null, max: null, variance: null };
  } else {
    // 计算最小值和最大值
    const min = Math.min(...validValues);
    const max = Math.max(...validValues);
    
    // 计算均值
    const sum = validValues.reduce((acc, val) => acc + val, 0);
    const mean = sum / validValues.length;
    
    // 计算方差
    const squaredDiffs = validValues.map(val => Math.pow(val - mean, 2));
    const varianceSum = squaredDiffs.reduce((acc, val) => acc + val, 0);
    const variance = varianceSum / validValues.length;
    
    // 更新统计信息，保留适当的小数位数
    factorStats.value = {
      mean: mean.toFixed(6),
      min: min.toFixed(6),
      max: max.toFixed(6),
      variance: variance.toFixed(6)
    };
  }
  
  // 同时计算TimeRange范围内的统计信息
  calculateTimeRangeStats(factor);
};

// 计算基于TimeRange的factor统计信息（忽略空值）
const calculateTimeRangeStats = (factor) => {
  if (!factor || csvData.value.length === 0) {
    timeRangeStats.value = { mean: null, min: null, max: null, variance: null };
    return;
  }
  
  // 确保时间范围有效
  const start = Math.max(0, Math.min(timeRange.value.start, csvData.value.length - 1));
  const end = Math.max(start, Math.min(timeRange.value.end, csvData.value.length - 1));
  
  // 获取时间范围内的数据
  const rangeData = csvData.value.slice(start, end + 1);
  
  // 过滤出有效的数值数据
  const validValues = rangeData
    .map(row => row[factor])
    .filter(value => value !== null && value !== undefined && value !== '' && typeof value === 'number' && !isNaN(value));
  
  if (validValues.length === 0) {
    timeRangeStats.value = { mean: null, min: null, max: null, variance: null };
    return;
  }
  
  // 计算最小值和最大值
  const min = Math.min(...validValues);
  const max = Math.max(...validValues);
  
  // 计算均值
  const sum = validValues.reduce((acc, val) => acc + val, 0);
  const mean = sum / validValues.length;
  
  // 计算方差
  const squaredDiffs = validValues.map(val => Math.pow(val - mean, 2));
  const varianceSum = squaredDiffs.reduce((acc, val) => acc + val, 0);
  const variance = varianceSum / validValues.length;
  
  // 更新统计信息，保留适当的小数位数
  timeRangeStats.value = {
    mean: mean.toFixed(6),
    min: min.toFixed(6),
    max: max.toFixed(6),
    variance: variance.toFixed(6)
  };
};

// 组件挂载时初始化图表
onMounted(() => {
  initChart();
});
</script>

<style scoped>
/* 滚动条样式 */
#factors ::-webkit-scrollbar {
  width: 6px;
}

#factors ::-webkit-scrollbar-track {
  background: #2a2b2d;
}

#factors ::-webkit-scrollbar-thumb {
  background: #555;
  border-radius: 3px;
}

#factors ::-webkit-scrollbar-thumb:hover {
  background: #777;
}

/* factor-select横向滚动条样式 - WebKit浏览器 */
#factor-select::-webkit-scrollbar {
  height: 6px;
  margin: 0 12px;
}
#factor-select::-webkit-scrollbar-track {
  background-color: #2a2b2d;
  border-radius: 3px;
}

#factor-select::-webkit-scrollbar-thumb {
  background: #555;
  border-radius: 3px;
  transition: background-color 0.2s ease;
}

#factor-select::-webkit-scrollbar-thumb:hover {
  background: #777;
}

#factor-select {  
  scrollbar-color: transparent transparent;
  transition: scrollbar-color 0.2s ease;
}
#factor-select:hover {  
  scrollbar-color: #333 transparent;
}

#factor-select {
  max-height: 50%;
}

/* 确保factors区域有滚动条 */
#factors .overflow-y-auto {
  max-height: 60%;
}

/* 调整时间选择器样式 */
#timeselect input[type="range"] {
  width: 100%;
  height: 4px;
  background: #333;
  outline: none;
  border-radius: 2px;
}

#timeselect input[type="range"]::-webkit-slider-thumb {
  appearance: none;
  width: 8px;
  height: 4px;
  background: #02DA7F;
  cursor: pointer;
}

/* 复选框样式 */
input[type="checkbox"] {
  width: 12px;
  height: 12px;
  cursor: pointer;
}
</style>