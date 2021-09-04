<template>
    <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
    <el-breadcrumb-item :to="{ path: '/home' }">Accueil</el-breadcrumb-item>
    <el-breadcrumb-item>Statistiques</el-breadcrumb-item>
    <el-breadcrumb-item>Liste des données</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
    <div ref= "chart" style ="width: 600px;height:400px;"></div>
    </el-card>
    </div>
</template>

<script>
import echarts from 'echarts'

export default {
    data(){
        return{}
    },

    created(){

    },
    async mounted(){

        this.initEchart()

        var myChart = echarts.init(document.getElementById('main'))

        const {data:result} = await this.$http.get('reports/type/1')


      myChart.setOption(option)
    },
    methods:{}
}
</script>

<style lang="less" scoped>

</style>
