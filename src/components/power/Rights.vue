<template>
    <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
    <el-breadcrumb-item :to="{ path: '/home' }">Accueil</el-breadcrumb-item>
    <el-breadcrumb-item>Gestion des droits</el-breadcrumb-item>
    <el-breadcrumb-item>Liste des droits</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
        <el-table :data= "rightsList" border stripe>
        <el-table-column type= "index"></el-table-column>
        <el-table-column label= "Nom des droits" prop="authName"></el-table-column>
        <el-table-column label= "Routes" prop="path"></el-table-column>
        <el-table-column label= "Grade" prop="level">
            <template slot-scope= "scope">
            <el-tag v-if= "scope.row.level === '0'">Niveau 1</el-tag>
            <el-tag type= "success" v-else-if= "scope.row.level === '1'">Niveau 2</el-tag>
            <el-tag type= "warning" v-else>Niveau 3</el-tag>
            </template>
        </el-table-column>
        </el-table>
    </el-card>

    </div>
</template>

<script>
export default {
    data() {
        return{
            rightsList: []
        }
    },
    created() {
        this.getRightsList()
    },
    methods: {
        async getRightsList() {
            const {data: result} = await this.$http.get('rights/list')
            if(result.meta.status !== 200) {
                return this.$message.error("Échec de l'obtention d'une liste de droits")
            }
            this.rightsList = result.data
            console.log(this.rightsList)
        }
    }
}
</script>

<style lang="less" scoped>
</style>
