# Vue3 + TypeScript实现文件上传功能

# 使用示例
<Upload 
    :fileTypeList="['xlsx', 'png', 'jpg', 'jepg', 'pdf', 'doc', 'docx']" 
    file-limit="20mb"
    :defaultFileList="fileList" 
    @update-files="updateFile" 
    @remove-files="removeFile" 
/>