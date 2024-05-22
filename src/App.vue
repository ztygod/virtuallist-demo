<template>
  <div id="app">
    <div class="virtualList">
      <h2>定高虚拟列表</h2>
      <VirtualListComponent :listData="virtualListData" :itemSize="100"></VirtualListComponent>
    </div>
    <div class="ESVirtualList">
      <h2>不定高虚拟列表</h2>
      <EsVirtualListComponent 
        :listData="EsvirtualListData" 
        :estimatedItemSize="100"
        v-slot="slotProps"
        >
        <MyItem :item="slotProps.item"/>
      </EsVirtualListComponent>
    </div>
  </div>
</template>


<script>
import VirtualListComponent from './components/VirtualListComponent.vue';
import EsVirtualListComponent from './components/EsVirtualListComponent.vue';
import MyItem from './components/Item.vue'
import faker from 'faker';

//定高虚拟列表
let virtualListData = []
for(let i = 0;i < 1000;i++) {
  virtualListData.push({id:i,value:i})
} 
//不定高虚拟列表
let EsvirtualListData =[]
for(let id = 0;id < 1000;id++){
  EsvirtualListData.push({
    id,
    value:faker.lorem.sentences()
  })
}
export default{
  name:'App',
  components:{
    VirtualListComponent,
    EsVirtualListComponent,
    MyItem
  },
  data() {
    return {
      virtualListData:virtualListData,
      EsvirtualListData:EsvirtualListData
    }
  },
}
 </script>

 <style>
.virtualList{
  height:600px;
  width:600px ;
  float: left;
}
.ESVirtualList{
margin-left: 650px;
  margin-top: 100px;
  width: 600px;
  height: 600px;
}
</style>

