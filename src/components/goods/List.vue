<template>
    <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
    <el-breadcrumb-item :to="{ path: '/home' }">Accueil</el-breadcrumb-item>
    <el-breadcrumb-item>La gestion des produits</el-breadcrumb-item>
    <el-breadcrumb-item>La liste des références</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
    <el-row :gutter= "20">
    <el-col :span= "8">
    <el-input placeholder= "Entrez quelque chose" v-model= "queryInfo.query" clearable @clear= "getGoodsList">
    <el-button slot="append" icon="el-icon-search" @click= "getGoodsList"></el-button>
  </el-input>
    </el-col>
    </el-col :span= "4">
    <el-button type= "primary" @click= "goAddpage">Ajouter un produit</el-button>
    </el-row>
    <el-table :data ="goodslist" border stripe>
    <el-table-column type= "index"></el-table-column>
    <el-table-column label= "Nom Des Produits" prop= "goods_name"></el-table-column>
    <el-table-column label= "Prix Des Produits" prop= "goods_price" width= "95px"></el-table-column>
    <el-table-column label= "Poids Des Produits" prop= "goods_weight" width= "70px"></el-table-column>
    <el-table-column label= "Création" prop= "add_time" width= "140px">
    <template slot-scope= "scope">
    {{scope.row.add_time | dateFormat}}
    </template>
    </el-table-column>
    <el-table-column label= "Opération" width= "130px">
    <template slot-scope= "scope">
    <el-button type="primary" icon="el-icon-edit" size= "mini"></el-button>
    <el-button type="danger" icon="el-icon-delete" size= "mini" @click= "removeById(scope.row.goods_id)"></el-button>
    </template>
    </el-table-column>
    </el-table>
     <el-pagination
      @size-change= "handleSizeChange"
      @current-change= "handleCurrentChange"
      :current-page.sync= "queryInfo.pagenum"
      :page-sizes="[5, 10, 15, 20]"
      :page-size= "queryInfo.pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total= "total" background>
    </el-pagination>
    </el-row>
    </el-card>
    </div>
</template>

<script>
export default {
    data(){
        return{
            queryInfo: {
                query: '',
                pagenum: 1,
                pagesize: 10
            },
            goodslist: [],
            total:0
        }
    },

    created() {
        this.getGoodList()
    },

    methods: {
        async getGoodList() {
            const {data:result} = await this.$http.get('goods', { params: this.queryInfo })
        
        if(result.meta.status !== 200) {
            return this.$message.error('Echec!')
        }
        this.$message.success('Succès!')
        console.log(result.data)
        this.goodslist = result.data.goods
        this.total = result.data.total
        },
        handleSizeChange(newSize){
            this.queryInfo.pagesize = newSize
            this.getGoodList()
        },
        handleCurrentChange(newPage){
            this.queryInfo.pagenum = newPage
            this.getGoodList()
        },
        async removeById(id){
        const confirmResult = await this.$confirm('Ceci effacera le fichier. Continuer?', 'Warning', {
          confirmButtonText: 'OK',
          cancelButtonText: 'Annuler',
          type: 'warning'
        }).catch(err => err)

        if(confirmResult !== 'confirm'){
            return this.$message.info('Annulé!')
        }
        const {data:result} = await this.$http.delete(`goods/${id}`)

        if(result.meta.status !==200){
            return this.$message.error('Echec!')
        }

        this.$message.success('Succès!')
        this.getGoodList()
        },
        goAddpage(){
            this.$router.push('/goods/add')
        }
    }
}
</script>

<style lang="less" scoped>

</style>
