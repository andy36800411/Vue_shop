<template>
    <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
    <el-breadcrumb-item :to="{ path: '/home' }">Accueil</el-breadcrumb-item>
    <el-breadcrumb-item>Gestion des commandes</el-breadcrumb-item>
    <el-breadcrumb-item>Liste des commandes</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
    <el-row>
    <el-col :span= "8">
    <el-input placeholder="Entrez quelque chose">
    <el-button slot="append" icon="el-icon-search"></el-button>
  </el-input>
    </el-col>
    </el-row>
    <el-table :data= "orderlist" border stripe>
    <el-table-column type= "index"></el-table-column>
    <el-table-column label= "Numéro des commandes" prop= "order_number"></el-table-column>
    <el-table-column label= "Prix des commandes" prop= "order_price"></el-table-column>
    <el-table-column label= "Payer?" prop= "pay_status">
    <template slot-scope= "scope">
    <el-tag type= "success" v-if= "scope.row.pay_status === '1'">Payé</el-tag>
    <el-tag type= "danger" v-else>Pas encore payé</el-tag>
    </template>
    </el-table-column>
    <el-table-column label= "Livrer?" prop= "is_send">
    <template slot-scope= "scope">{{scope.row.is_send}}</template>
    </el-table-column>
    <el-table-column label= "Passer une commande?" prop= "create_time">
    <template slot-scope= "scope">{{scope.row.create_time | dateFormat}}</template>
    </el-table-column>
    <el-table-column label= "Opération">
    <template slot-scope= "scope">
    <el-button type="primary" icon="el-icon-edit" size= "mini" @click= "showBox"></el-button>
    <el-button type="success" icon="el-icon-location" size= "mini" @click= "showProgressBox"></el-button>
    </template>
    </el-table-column>
    </el-table>
    <el-pagination
      @size-change= "handleSizeChange"
      @current-change= "handleCurrentChange"
      :current-page.sync= "queryInfo.pagenum"
      :page-sizes= "[5, 10, 15]"
      :page-size= "queryInfo.pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total= "total">
    </el-pagination>
    </el-card>

    <el-dialog
  title="Modifier l'adresse"
  :visible.sync= "addressVisible"
  width="50%" @close= "addressDialogClosed">
  <el-form :model= "addressForm" :rules= "addressFormRules" ref="addressFormRef" label-width="155px">
  <el-form-item label= "Province, ville, district" prop="address1">
    <el-cascader :options= "cityData" v-model= "addressForm.address1"></el-cascader>
  </el-form-item>
  <el-form-item label= "L'address exacte" prop="address2">
    <el-input v-model= "addressForm.address2"></el-input>
  </el-form-item>
  </el-form>
  <span slot="footer" class="dialog-footer">
    <el-button @click= "addressVisible = false">Annuler</el-button>
    <el-button type="primary" @click= "addressVisible = false">Confirmer</el-button>
  </span>
</el-dialog>

<el-dialog
  title= "Progrès de la logistique"
  :visible.sync= "progressVisible"
  width="50%" @close= "addressDialogClosed">
  <el-timeline>
    <el-timeline-item
      v-for="(activity, index) in progressInfo"
      :key="index"
      :timestamp="activity.time">
      {{activity.context}}
    </el-timeline-item>
  </el-timeline>
</el-dialog>
    </div>
</template>

<script>
import cityData from './citydata.js'

export default {
    data(){
        return{
            queryInfo:{
                query: '',
                pagenum: 1,
                pagesize: 10
            },
            total: 0,
            orderlist:[],
            addressVisible:false,
            addressForm:{
                address1: [],
                address2: ''
            },
            addressFormRules:{
                address1:[
                { required: true, message: "Veuillez enter votre province, ville, district", trigger: 'blur' },
                ],
                address2:[
                { required: true, message: "Veuillez enter votre L'address exacte", trigger: 'blur' },
                ],
            },
            cityData,
            progressVisible: false,
            progressInfo:[]
        }
    },
    created(){
        this.getOrderList()
    },
    methods: {
        async getOrderList(){
            const {data:result} = await this.$http.get('orders', {params: this.queryInfo})

            if(result.meta.status !==200){
                return this.$message.error('Ehcec')
            }

            this.total = result.data.total
            this.orderlist = result.data.goods
        },
        handleSizeChange(newSize){
            this.queryInfo.pagesize = newSize
            this.getOrderList()
        },
        handleCurrentChange(newPage){
            this.queryInfo.pagenum = newPage
            this.getOrderList()
        },
        showBox(){
            this.addressVisible = true
        },
        addressDialogClosed(){
          this.$refs.addressFormRef.resetFields()  
        },
        async showProgressBox(){
            const {data:result} = await this.$http.get('/kuaidi/:id/804909574412544580')

            if(result.meta.status !==200){
                return this.$message.error('Echec!')
            }

            this.progressInfo = result.data
            this.addressVisible = true
            
            console.log(this.progressInfo)

        }
    }
}
</script>

<style lang="less" scoped>
@import '../../plugins/timeline/timeline.css';
@import '../../plugins/timeline-item/timeline-item.css';

.el-cascader{
    width: 100%;
}

</style>
