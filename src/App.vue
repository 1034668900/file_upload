<script setup>
import { onMounted, reactive } from "vue";

// 获取DOM对象并监听文件/文件夹选择
let fileDragArea = null;
let selectFileEle = null;
let selectDirectoryEle = null;
// 选择的所有文件
let files = [];
let tableData = reactive([]);
onMounted(() => {
  // 获取对象
  fileDragArea = document.querySelector("#fileDragArea");
  selectFileEle = document.querySelector("#selectFile");
  selectDirectoryEle = document.querySelector("#selectDirectory");
  // 注册事件
  selectFileEle.onchange = () => {
    console.log("拿到file对象", selectFileEle.files);
    let fileList = [...selectFileEle.files];
    fileList.forEach((file) => {
      files.push(file);
      setTableData(file.name, "暂无路径");
    });
  };
  selectDirectoryEle.onchange = () => {
    console.log("拿到文件夹内所有file对象", selectDirectoryEle.files);
    let fileList = [...selectDirectoryEle.files];
    fileList.forEach((file) => {
      files.push(file);
      setTableData(file.name);
    });
  };
  // 为拖拽区域注册事件 -- div默认无法实现文件拖拽，因此必须阻止默认行为
  // 进入拖拽区域触发
  fileDragArea.ondragenter = (e) => {
    e.preventDefault();
  };
  // 在拖拽区域会一直触发
  fileDragArea.ondragover = (e) => {
    e.preventDefault();
  };
  // 在拖拽区域松开鼠标左键触发
  fileDragArea.ondrop = (e) => {
    e.preventDefault();
    for (const item of e.dataTransfer.items) {
      // 拿到句柄
      const entry = item.webkitGetAsEntry();
      console.log("entry", entry);
      if (entry.isDirectory) {
        // 目录
        readDirectory(entry, files);
      } else {
        // 文件
        entry.file((file) => {
          console.log("成功拿到file对象", file);
          setTableData(item.name, item.fullPath);
          files.push(file);
        });
      }
    }
    console.log("本次拖拽的所有文件", files, tableData);
  };
});

// 目录读取
function readDirectory(directory, entries = []) {
  //DirectoryEntry接口的createReader方法可以创建一个FileSystemDirectoryReader用于读取目录
  let dirReader = directory.createReader();
  dirReader.readEntries((results) => {
    if (results.length) {
      results = [...results];
      results.forEach((item) => {
        if (item.isFile) {
          setTableData(item.name, item.fullPath);
          entries.push(item);
        } else {
          // 递归
          readDirectory(item, entries);
        }
      });
    }
  });
  return entries;
}

// 设置TableData信息
function setTableData(name, path = "暂无路径") {
  tableData.push({
    name,
    fullPath: path,
  });
}
</script>

<template>
  <div>
    <div id="fileDragArea">文件上传区域(支持拖拽)</div>
    <el-button @click="selectFileEle.click()" class="selectBtn" type="primary"
      >选择文件</el-button
    >
    <el-button
      @click="selectDirectoryEle.click()"
      class="selectBtn"
      type="primary"
      >选择文件夹</el-button
    >
    <input id="selectFile" style="display: none" type="file" multiple />
    <input
      id="selectDirectory"
      style="display: none"
      type="file"
      multiple
      webkitdirectory
      mozdirectory
      odirectory
    />
    <el-table :data="tableData" style="width: 100%;height: 400px;">
      <el-table-column prop="name" label="文件名" width="280" />
      <el-table-column prop="fullPath" label="路径" />
    </el-table>
  </div>
</template>

<style scoped>
#fileDragArea {
  width: 100%;
  height: 200px;
  border: 1px solid #000;
  display: flex;
  justify-content: center;
  align-items: center;
}
.selectBtn {
  margin-top: 20px;
}
</style>
