<template>
  <div class="file-manager-container h-[4%] flex w-full">
    <!-- 文件列表显示区域 -->
    <div id="file-select" class="max-w-full overflow-y-hidden overflow-x-scroll h-full w-[99%] text-xs flex items-center justify-start my-[1%] text-gray-500 rounded">
      <div 
        v-for="fileData in filesData" 
        :key="fileData.id"
        @click="switchFile(fileData.id)"
        class="text-ellipsis cursor-pointer flex items-center text-gray-400 truncate bg-[#333] h-full text-xs rounded px-1 mx-1 hover:text-gray-100 transition-colors duration-200"
        :class="{ 
          'bg-[#02DA7F] text-white': fileData.id === activeFileId, 
          'opacity-50': fileData.status === 'error' || fileData.status === 'loading'
        }"
      >
        <p class="truncate">{{ fileData.name }}</p>
        <div 
          @click.stop="removeFile(fileData.id)"
          class="flex p-0 items-center justify-center text-center ml-2 text-white bg-[#333] rounded-full aspect-square cursor-pointer h-[70%] text-xs hover:bg-[#d15858] duration-200"
        >
          ×
        </div>
        <span v-if="fileData.status === 'loading'" class="ml-1 text-[#02b875]">加载中...</span>
        <span v-else-if="fileData.status === 'error'" class="ml-1 text-[#d15858]">错误</span>
      </div>
      
      <!-- 添加文件按钮 -->
      <div 
        id="addfile" 
        @click="triggerFileUpload"
        class="flex items-center justify-center text-center mx-2 text-white bg-[#333] rounded aspect-square cursor-pointer h-[95%] text-xs hover:bg-[#02b875] duration-200"
      >
        +
      </div>
      
      <!-- 隐藏的文件输入框 -->
      <input 
        ref="fileInput"
        type="file" 
        accept=".csv" 
        class="hidden"
        multiple
        @change="handleFileSelect"
      >
    </div>
  </div>
</template>

<script setup>
import { ref, defineEmits, defineProps } from 'vue';

// 定义属性
const props = defineProps({
  activeFileId: {
    type: String,
    default: null
  }
});

// 定义事件
const emit = defineEmits([
  'file-added',    // 文件添加时触发
  'file-removed',  // 文件移除时触发
  'file-switched'  // 文件切换时触发
]);

// 文件状态管理
const fileInput = ref(null);
const filesData = ref([]);

// 触发文件上传
const triggerFileUpload = () => {
  fileInput.value?.click();
};

// 处理文件上传
const handleFileSelect = (event) => {
  const files = event.target.files;
  for (let i = 0; i < files.length; i++) {
    const file = files[i];
    const fileId = generateUniqueId();
    
    // 检查文件名是否已存在
    if (filesData.value.some(f => f.name === file.name)) {
      // 如果存在，添加时间戳后缀
      const newName = file.name + ' (' + new Date().toLocaleTimeString() + ')';
      
      filesData.value.push({
        id: fileId,
        name: newName,
        file: file,
        status: 'loading'
      });
      
      emit('file-added', {
        id: fileId,
        name: newName,
        file: file
      });
    } else {
      filesData.value.push({
        id: fileId,
        name: file.name,
        file: file,
        status: 'loading'
      });
      
      emit('file-added', {
        id: fileId,
        name: file.name,
        file: file
      });
    }
  }
  
  // 清空input以允许重新选择同一文件
  event.target.value = '';
};

// 切换文件
const switchFile = (fileId) => {
  if (fileId !== props.activeFileId && filesData.value.find(f => f.id === fileId)?.status === 'loaded') {
    emit('file-switched', fileId);
  }
};

// 移除文件
const removeFile = (fileId) => {
  const index = filesData.value.findIndex(file => file.id === fileId);
  if (index !== -1) {
    filesData.value.splice(index, 1);
    emit('file-removed', fileId);
    
    // 如果移除的是当前激活的文件，且还有其他文件，自动切换到第一个文件
    if (props.activeFileId === fileId && filesData.value.length > 0) {
      emit('file-switched', filesData.value[0].id);
    }
  }
};

// 更新文件状态（供父组件调用）
const updateFileStatus = (fileId, status, data = {}) => {
  const fileData = filesData.value.find(file => file.id === fileId);
  if (fileData) {
    fileData.status = status;
    // 合并其他文件数据
    Object.assign(fileData, data);
  }
};

// 生成唯一ID
const generateUniqueId = () => {
  return Date.now().toString(36) + Math.random().toString(36).substr(2);
};

// 暴露方法给父组件
defineExpose({
  filesData,
  updateFileStatus
});
</script>

<style scoped>

#file-select::-webkit-scrollbar {
  height: 6px;
  margin: 0 12px;
}
#file-select::-webkit-scrollbar-track {
  background-color: #2a2b2d;
  border-radius: 3px;
}

#file-select::-webkit-scrollbar-thumb {
  background: #555;
  border-radius: 3px;
  transition: background-color 0.2s ease;
}

#file-select::-webkit-scrollbar-thumb:hover {
  background: #777;
}

#file-select {  
  scrollbar-color: transparent transparent;
  transition: scrollbar-color 0.2s ease;
}
#file-select:hover {  
  scrollbar-color: #333 transparent;
}

#factor-select {
  max-height: 80%;
}
</style>