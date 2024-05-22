<template>
    <p>
        <span style="color: red">{{ item.id }}</span>
        {{ item.value }}
    </p>
</template>

<script>
export default{
    name:'MyItem',
    props:{
        item:{
            type:Object,
            default:() => {}
        }
    }
}
</script>