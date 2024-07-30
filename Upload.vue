<template>
    <div class="Upload">
        <div class="upload-top">
            <div class="upload-btn" @click="selectFile">点击上传</div>
            <div class="upload-tip">支持{{ fileTypeList.join('、') }}等常规可预览文件格式</div>
        </div>
        <div class="file-list">
            <div class="file-item" v-for="item in fileList" :key="item.fileId">
                <div class="file-name">
                    <img src="@/assets/images/file.png" alt="">
                    {{ item.fileName }}
                </div>
                <div class="file-close">
                    <img src="@/assets/images/fileClose.png" alt="" @click="removeFile(item)">
                </div>
            </div>
        </div>
    </div>
</template>
<script lang="ts" setup>
import { v4 as uuid } from 'uuid'
type Props = {
    defaultFileList?: any[], //用于回显文件
    fileTypeList: any[], //文件类型限制
    fileLimit: string, //100B || 100KB || 100MB || 100GB
    countLimit?: number, //上传数量限制
}
const props = defineProps<Props>()
const emit = defineEmits(['update-files', 'remove-files'])
const sizeTypeList = ['B', 'KB', 'MB', 'GB']
const fileList = ref<any[]>([])
const selectFile = () => {
    const input = document.createElement('input')!
    input.type = 'file'
    input.click()
    input.addEventListener('change', (e: any) => {
        const file = e.target.files[0]
        const fileType = file.name.split('.').pop()
        const fileSize = file.size
        const fileName = file.name
        //检查文件类型
        if (!checkFileType(fileType)) return ElMessage.error('文件类型不符合！')
        //检查文件大小是否合规
        if (!checkFileSize(fileSize)) return ElMessage.error('文件尺寸不符合！')
        if (props.countLimit && fileList.value.length + 1 > props.countLimit) return ElMessage.error('超过上传限制数量！')
        fileList.value.push({
            fileType,
            fileId: 'new-' + uuid(), //新增文件加new-标识
            fileName: `${fileName}（${getFileSize(fileSize)}）`,
            fileSize: getFileSize(fileSize),
            file: e.target.files[0]
        })
        emit('update-files', fileList.value)
    })

}
const checkFileType = (fileType: string) => {
    if (props.fileTypeList.includes(fileType)) return true
    return false
}
const checkFileSize = (fileSize: number) => {
    const fileType = props.fileLimit.match(/[^0-9]+/g)?.pop()?.toLocaleUpperCase()
    const fileLimit = Number.parseInt(props.fileLimit)
    //将设置的文件最大尺寸转换为byte
    let maxLimit;
    if (sizeTypeList.indexOf(fileType as string) > 0) {
        maxLimit = fileLimit * Math.pow(1024, sizeTypeList.indexOf(fileType as string))
    } else {
        maxLimit = fileLimit
    }
    if (fileSize < maxLimit) return true;
    return false
}

//移出文件
const removeIds = ref<string[]>([])
const removeFile = (row: any) => {
    const newList = fileList.value.filter((item: any) => item.fileId !== row.fileId)
    fileList.value = newList
    emit('update-files', fileList.value)
    //对比文件，是新上传的不传移出的文件id
    if (row.fileId.slice(0, 4) === 'new-') return;
    removeIds.value.push(row.fileId)
    emit('remove-files', removeIds.value)
}
//文件回显
watch(() => props.defaultFileList, (val) => {
    if (val) {
        fileList.value = val
    }
}, { immediate: true })

//获取文件大小
const getFileSize = (size: number) => {
    //b
    if (size < 1024) return size + 'b';
    //kb
    if (size / 1024 / 1024 < 1) return (size / 1024).toFixed(2) + 'Kb';
    //Mb
    if (size / 1024 / 1024 / 1024 < 1) return (size / 1024 / 1024).toFixed(2) + 'Mb';
}

</script>
<style lang="scss">
.Upload {
    .upload-top {
        display: flex;
        align-items: center;

        .upload-btn {
            width: 72px;
            height: 32px;
            background: #6089F2;
            border-radius: 8px;
            display: flex;
            align-items: center;
            justify-content: center;
            font-weight: 400;
            font-size: 14px;
            color: #FFFFFF;
            cursor: pointer;
        }

        .upload-tip {
            font-weight: 400;
            font-size: 14px;
            color: #999999;
            margin-left: 12px;
        }
    }

    .file-list {
        width: 480px;

        .file-item {
            width: 100%;
            padding: 8px 12px;
            box-sizing: border-box;
            background-color: #F5F7FA;
            margin-top: 12px;
            display: flex;
            align-items: center;
            justify-content: space-between;

            .file-name {
                display: flex;
                align-items: center;
                font-weight: 400;
                font-size: 14px;
                color: #6089F2;
                line-height: 16px;

                img {
                    width: 14px;
                    height: 16px;
                    margin-right: 10px;
                }
            }

            .file-close {
                width: 12px;
                height: 12px;
                display: flex;
                align-items: center;
                justify-content: center;

                img {
                    width: 100%;
                    height: 100%;
                    cursor: pointer;
                }
            }
        }
    }
}
</style>