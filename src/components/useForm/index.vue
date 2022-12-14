<template>
    <a-form 
        :model="formParam" 
        ref="formRef" name="formRef" 
        :label-col="labelCol" 
        :wrapper-col="wrapperCol"
    >
        <a-row :gutter="gutter">
            <template v-for="item in formColumns" :key="item.prop">
                <a-col :span="span">
                    <a-form-item v-if="item.isHasShow" :label="`${item.title} :`" :rules="item?.rule" :name="item['value']">
                        <useFormItem v-if="item.componentType === 'default'" :formParam="formParam" :item="item"/>
                        <!-- 按需插槽 -->
                        <slot v-if="item.componentType === 'slot'" :name="`${item['value']}FormItem`" :row="formParam"></slot>
                        <!-- 自定义模板渲染 -->
                        <component v-if="item.componentType === 'component' && item?.renderForm" :is="item?.renderForm" :row="formParam"></component>
                    </a-form-item>
                </a-col>
            </template>
        </a-row>
        <!-- 自定义插入内容 -->
        <slot name="content"></slot>
    </a-form>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import useFormItem from './components/useFormItem.vue';
import { FormInstance } from 'ant-design-vue';

interface useForm {
    columns:useFormProps[], //所有表单项
    initFormParam?:any, //初始化form数据
    submitApi:(params:any)=> Promise<any>, //提交表单api
    labelCol?:any, //label 标签布局
    wrapperCol?:any, //控件设置布局样式
    gutter?:number, //栅格间隔
    span?:number, //格占位格数
}

// 默认值
const props = withDefaults(defineProps<useForm>(),{
    columns:()=>[],
    initFormParam:{},
    labelCol:{ span: 4 },
    wrapperCol:{ span: 20 },
    gutter:16,
    span:12
})

// 设置form表单数据
const formParam = ref<any>({})
// 重置数据
const setupFormData = ()=>{
    props.columns.forEach(item=>{
        formParam.value[item.value] = null
    })
    formParam.value = Object.assign(formParam.value,props.initFormParam)
}

setupFormData()

 //表单配置项
const formColumns = props.columns.map((item):useFormProps=>{
    return {
        isHasShow:true,
        clearable:true,
        componentType:'default',
        ...item
    }
})

// 表单
const formRef = ref<FormInstance>()
// 提交
const submitForm = async ()=>{
    await formRef.value.validate() //先进行表单验证
    await props.submitApi(formParam.value)
}

// 重置
const reset = (type: 'reset' | 'clear')=>{
    if(!formRef.value) return
    if(!type){
        formRef.value.clearValidate()
        formRef.value.resetFields()
    } else if (type === 'reset') {
        formRef.value.resetFields()
    } else {
        formRef.value.clearValidate()

    }
}

defineExpose({ formParam, submitForm, reset })

</script>

<style scoped>

</style>