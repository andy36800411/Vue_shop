<template>
    <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
    <el-breadcrumb-item :to="{ path: '/home' }">Accueil</el-breadcrumb-item>
    <el-breadcrumb-item>La gestion des produits</el-breadcrumb-item>
    <el-breadcrumb-item>La liste des références</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
    <el-alert show-icon title= "Seuls les paramètres pertinents sont autorisés pour les catégories niveau 3 !"
    type= "warning" :closable= "false"></el-alert>
    <el-row class= "cat_opt">
    <el-col>
    <span>Sélectionnez la catégorie des produits: </span>
    <el-cascader
    v-model= "selectedCateKeys"
    :options= "catelist"
    :props = cateProps
    expand-trigger= "hover"
    @change= "handleChange"></el-cascader>
    </el-col>
    </el-row>

    <el-tabs v-model= "activeName" @tab-click= "handleTabClick">
    <el-tab-pane label= "Paramètres dynamiques" name="many">
    <el-button type= "primary" size= "mini" :disabled= "isBtnDisabled" @click= "addDialogVisible=true">Ajout de paramètres</el-button>
    <el-table :data= "onlyTableData" border stripe></el-table>
    <el-table-column type= "expand"></el-table-column>
    <el-table-column type= "index"></el-table-column>
    <el-table-column label= "Nom de paramètre" prop= "attr_name"></el-table-column>
    <el-table-column label= "Opération">
    <template slot-scope= "scope">
    <el-button size="mini" type="primary" icon="el-icon-edit" @click= "showEditDialog">Editer</el-button>
    <el-button size="mini" type="danger" icon="el-icon-delete" >Supprimer</el-button>
    </template>
    </el-table-column>
    </el-tab-pane>
    <el-tab-pane label= "Propriétés statiques" name="only">
    <el-button type= "primary" size= "mini" :disabled= "isBtnDisabled" @click= "addDialogVisible=true">Ajout de propriétés</el-button>
    <el-button size="mini" type="danger" icon="el-icon-delete" >Supprimer</el-button>
    </el-tab-pane>
    </el-tabs>
    </el-card>

    <el-dialog
  :title= "'Ajouter' + titleText"
  :visible.sync= "addDialogVisible"
  width="50%" @close= "addDialogClosed">
  <el-form :model= "addForm" :rules= "addFormRules" ref="addFormRef" label-width="170px">
  <el-form-item :label= "titleText" prop= "attr_name">
    <el-input v-model= "addForm.attr_name"></el-input>
  </el-form-item>
  </el-form>
  <span slot="footer" class="dialog-footer">
    <el-button @click= "addDialogVisible = false">Annuler</el-button>
    <el-button type="primary" @click= "addParams">Confirmer</el-button>
  </span>
</el-dialog>
    </div>
</template>

<script>
export default {
    data() {
      return {
        catelist: [],
        cateProps: {
            value: 'cat_id',
            label: 'cat_name',
            children: 'children'
        },
        selectedCateKeys: [],
        activeName: 'many',
        manyTableData: [],
        onlyTableData: [],
        addDialogVisible: false,
        addForm: {
            attr_name: ''
        },
        addFormRules:{
            attr_name: [
                { required: true, message: "Veuillez enter le nom du paramètre", trigger: 'blur' },
                ],
        }

      }  
    },
    created() {
    this.getCateList()
    },
    methods: {
    async getCateList(){
        const {data:result} = await this.$http.get('categories')
        if(result.meta.status !==200) {
            return this.$message.error("Échec de l'obtention la catégorie des produits")
        }

        this.catelist = result.data

        console.log(this.catelist)
    },
    handleChange() {
        this.getParamsData()
    },
    handleTabClick() {
        console.log(this.activeName)
        this.getParamsData()
    },
    async getParamsData() {
            if(this.selectedCateKeys.length !==3){
            this.selectedCateKeys = []
            return
        }

        console.log(this.selectedCateKeys)
        const {data:result} = await this.$http.get(`categories/${this.cateId}/attributed`, {
            params: {sel: this.activeName}})

            if(result.meta.status !==200){
                return this.$message.error('Echec')
            }
            result.data.forEach(item =>{
                item.attr_vals = item.attr_vals.split('')
            })
            console.log(result.data)
            if(this.activeName === 'many') {
                this.manyTableData = result.data
            } else {
                this.onlyTableData = result.data
            }
        }
    },
    addDialogClosed() {
        this.$refs.addFormRef.resetFields()
    },
    addParams(){
       this.$refs.addFormRef.validate(async valid => 
       {
           if(!valid) return
           const {data:result} = await this.$http.post(`categories/${this.cateId}/attributes`, {
               attr_name: 'this.addForm.attr_name',
               attr_sel: 'this.activeName'
           })

           if(result.meta.status !==201){
               return this.$message.error('Echec!')
           }

           this.$message.success('Succès')
           this.addDialogVisible = false
           this.getParamsData()
       }) 
    },
    computed: {
        isBtnDisabled() {
            if(this.selectedCateKeys.length !==3){
                return true
            }
            return false
        },

        cateId(){
            if(this.selectedCateKeys.length ===3){
                return this.selectedCateKeys[2]
            }
            return null
        },
        titleText() {
            if(this.activeName ==='many'){
                return 'Paramètres dynamiques'
            }
            return 'Propriétés statiques'
        }

    }
}
</script>

<style lang="less" scoped>

.cat_opt{
    margin: 15px 0px;
}

</style>
