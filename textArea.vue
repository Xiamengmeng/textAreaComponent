<template>
<section>
    <div class="position clearfix">
        <textarea class="bigBox" rows="10" cols="65" :placeholder="placeholder" :disabled='disabled' :maxLength='maxLength' @input="desInput" v-model="inputDes">
            
        </textarea>
        <div class="smallContent"><span v-if="count>=maxLength" class="red">文本框最大输入长度为{{maxLength}}字符</span>{{count}}/{{maxLength}}</div>
    </div>
</section>
</template>
<script>
export default {
    name:"textArea",
    data(){
        return {
            inputDes:'',
            count:0
        }
    },
    props:{
        placeholder:{
            type:String,
            default:'请输入'
        },
        maxLength:{
            type:Number,
            default:1000
        },
        disabled:{
            type:Boolean,
            default:false
        }
    },
    methods:{
       desInput(){
           this.$emit('input',this.inputDes)
           this.count=this.inputDes.length
       }
    },
    watch:{
        
    }
}
</script>
<style lang='less' scoped>
@import '../assets/css/vars';
.clearfix{
    overflow: hidden;
}
.position{
    position: relative;
}
.red{
    color:@fail-color;
}
.bigBox{
    padding:5px;
    resize:none;
    width:99%;
    position: relative;
}
.smallContent{
    position: absolute;
    right: 40px;
    bottom: 30px;
}
textarea:disabled{
    background-color: @disabled-color
}
</style>
