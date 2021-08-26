<template>
    <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
    <el-breadcrumb-item :to="{ path: '/home' }">Accueil</el-breadcrumb-item>
    <el-breadcrumb-item>La gestion des produits</el-breadcrumb-item>
    <el-breadcrumb-item>Catégories de produits</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
        <el-row>
        <el-col>
        <el-button type= "primary" @click= "showAddCateDialog">Ajouter une catégorie</el-button>
        </el-col>
        </el-row>
        <tree-table class= "treeTable" :data= "catelist" :columns= "columns" 
        :selection-type= "false" 
        :expand-type= "false" show-index index-text= "#" border :show-row-hover= "false">
        <template slot= "est ok" slot-scope= "scope">
            <i class="el-icon-success" v-if= "scope.row.cat_deleted === false" style= "color:lightgreen;"></i>
            <i class="el-icon-error" v-else style= "color:red;"></i>
        </template>
        <template slot= "order" slot-scope= "scope">
        <el-tag size= "mini" v-if= "scope.row.cat_level===0">Niveau 1</el-tag>
        <el-tag type= "success" size= "mini" v-else-if= "scope.row.cat_level===1">Niveau 2</el-tag>
        <el-tag type= "warning" size= "mini" v-else>Niveau 3</el-tag>
        </template>
        <template slot= "opt" slot-scope= "scope">
        <el-button type="primary" icon="el-icon-edit" size= "mini">Editer</el-button>
        <el-button type="danger" icon="el-icon-delete" size= "mini">Rechercher</el-button>
        </template>
        </tree-table>
        <el-pagination
      @size-change= "handleSizeChange"
      @current-change= "handleCurrentChange"
      :current-page= "querInfo.pagenum"
      :page-sizes="[3, 5, 10, 15]"
      :page-size= "querInfo.pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total= "total">
    </el-pagination>
    </el-card>
    <el-dialog
  title= "Ajouter une catégorie"
  :visible.sync= "addCateDialogVisible"
  width="50%" @close= "addCateDialogClosed">
    <el-form :model= "addCateForm" :rules= "addCateFormRules" ref="addCateFormRef" label-width="140px">
    <el-form-item label="Nom de catégorie:" prop="cat_name">
    <el-input v-model= "addCateForm.cat_name"></el-input>
    </el-form-item>
    <el-form-item label="Catégories plus élevé:">
    <el-cascader
    v-model= "selectedKeys"
    expand-trigger= "hover"
    :options= "parentCateList"
    :props= "cascaderProps"
    @change= "parentCateChanged" clearable change-on-select=""></el-cascader>
    </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
    <el-button @click= "addCateDialogVisible = false">Supprimer</el-button>
    <el-button type="primary" @click= "addCateDialogVisible = false">Valider</el-button>
  </span>
</el-dialog>
    </div>
</template>

<script>
export default {
    data() {
        return {
            querInfo: {
                type: 3,
                pagenum: 1,
                pagesize: 5
            },
            catelist: [],
            total: 0,
            columns: [
                {
                label: 'Nom de la catégorie',
                prop: 'cat_name'
                }, 
                {
                label: 'Est-il valable?',
                type: 'template',
                template: 'est ok'
                },
                {
                label: 'Order',
                type: 'template',
                template: 'order'
                },
                {
                label: 'Opération',
                type: 'template',
                template: 'opt'
                },
            ],
            addCateDialogVisible: false,
            addCateForm: {
                cat_name: '',
                cat_pid: 0,
                cat_level: 0
            },
            addCateFormRules: {
                cat_name: [
                { required: true, message: 'Veuillez enter nom de catégorie', trigger: 'blur' },
                ]
            },
            parentCateList: [],
            cascaderProps: {
                value: 'cat_id',
                label: 'cat_name',
                children: 'children'
            },
            selectedKeys:[]
        }
    },
    created() {
        this.getCateList()
    },
    methods: {
        async getCateList() {
            const {data:result} = await this.$http.get('categories', { params: this.querInfo })

            if (result.meta.status !==200){
                return this.$message.error('Échec de la récupération catégories de produits!')
            }

            this.catelist = result.data.result
            this.total = result.data.total
        },
        handleSizeChange(newSize) {
            this.querInfo.pagesize = newSize
            this.getCateList()
        },
        handleCurrentChange(newPage) {
            this.querInfo.pagenum = newPage
            this.getCateList()
        },
        showAddCateDialog() {
            this.getParentCateList()
            this.addCateDialogVisible = true
        },
        async getParentCateList() {
            const {data:result} = await this.$http.get('categories', {params: { type: 2 }})

            if(result.meta.status !==200) {
                return this.$message.error('Échec de la récupération des catégories plus élevé!')
            }

            this.parentCateList = result.data
        },
        parentCateChanged() {
            console.log(this.selectedKeys)

            if(this.selectedKeys.length >0) {
                this.addCateForm.cat_pid = this.selectedKeys[
                this.selectedKeys.this.selectedKeys.length -1
            ]
            this.addCateForm.cat_level = this.selectedKeys.length
            return
            } else {
            this.addCateForm.cat_pid = 0
            this.addCateForm.cat_level = 0
        }
        },
        addCate() {
           this.$refs.addCateFormRef.validate(async valid => {
               if(!valid) return
               const {data:result} = await this.$http.post('categories', this.addCateForm)

               if(result.meta.status !==201){
                   return this.$message.error('Ajouter un catégorie!')
               }

               this.$message.success("Succès de l'ajout un catégorie ")
               this.getCateList()
               this.addCateDialogVisible = false
           })
        },
        addCateDialogClosed() {
        this.$refs.addCateFormRef.resetFields()
        this.selectedKeys = []
        this.addCateForm.cat_level = 0
        this.addCateForm.pid = 0 
        }
    }
}
</script>>

<style lang="less" scoped>

.treeTable{
    margin-top:15px;
}

.el-cascader{
    width: 100%;
}
</style>
