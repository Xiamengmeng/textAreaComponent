<template>
    <el-upload
            :action="actionUrl"
            :limit="3"
            :file-list="picList"
            list-type="picture-card"
            :on-remove="handleFileRemove"
            :on-change="handleFileChange"
            :disabled="disabled">
        <i class="el-icon-plus"></i>
    </el-upload>
</template>

<script>
    import Utils from '../utils/utils'
    export default {
        name: "upload-img",
        props: {
            imgList:{
                type:Array,
                default:function () {
                    return []
                }
            },
            sign:{  //标记字符，通过emit再返给父组件
                type:String,
                default:''
            },
            disabled:{
                type:Boolean,
                default:false
            }
        },
        data(){
            return {
                actionUrl: '/api-aegean/zuul/investment/e/upload/attach',
                picList:[]
            }
        },
        methods:{
            //删除图片
            handleFileRemove(file, fileList) {
                const vm = this
                // console.log('remove file =>', {...file});
                const temp = vm.picList.filter(item=>{
                    return item.url !=file.url
                })
                this.picList = temp
                this.$emit('getPicList',{picList:vm.picList,sign:vm.sign})
            },
            //上传图片
            handleFileChange(file, fileList) {
                const vm = this
                if (file.status === 'success') {
                    // console.log('file =>', {...file});
                    vm.picList.push(Utils.formatViewFileVo(file))
                }
                this.$emit('getPicList',{picList:vm.picList,sign:vm.sign})
            },


        },
        created(){
        },
        watch:{
            imgList(n,o){
                if(this.imgList && this.imgList.length>0){
                    this.picList = this.imgList
                    //name和url必须有，用于展示图片
                    this.picList.forEach(item=>{
                        item.url = item.fileUrl
                        item.name = item.fileName
                    })
                }
            }
        }
    }
</script>

<style scoped>

</style>