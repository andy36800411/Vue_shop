<template>
    <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
    <el-breadcrumb-item :to="{ path: '/home' }">Accueil</el-breadcrumb-item>
    <el-breadcrumb-item>La gestion des produits</el-breadcrumb-item>
    <el-breadcrumb-item>L'ajout des produits</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
    <el-alert
    title= "L'ajout des informations sur le produit"
    type= "info"
    center
    show-icon :closable= "false">
  </el-alert>
  <el-steps :space="200" :active= "activeIndex - 0" finish-status="success" align-center>
  <el-step title="Information"></el-step>
  <el-step title="Paramètre des produits"></el-step>
  <el-step title="Attribut des produits"></el-step>
  <el-step title="Photo des produits"></el-step>
  <el-step title="Contenu des produits"></el-step>
  <el-step title="Terminé"></el-step>
</el-steps>
<el-form :model= "addForm" :rules= "addFormRules" ref="addFormRef" label-width="120px" label-position= "top">
<el-tabs v-model= "activeIndex" :tab-position= "'left'" :before-leave= "beforeTableLeave" @tab-click= "tableClicked">
    <el-tab-pane label= "Information" name= "0">
    <el-form-item label= "Nom du produit" prop= "goods_name">
    <el-input v-model= "addForm.goods_name"></el-input>
    </el-form-item>
     <el-form-item label= "Prix du produit" prop= "goods_price">
    <el-input v-model= "addForm.goods_price" type= "number"></el-input>
    </el-form-item>
     <el-form-item label= "Poids du produit" prop= "goods_weight">
    <el-input v-model= "addForm.goods_weight"></el-input>
    </el-form-item>
     <el-form-item label= "Quantité du produit" prop= "goods_number">
    <el-input v-model= "addForm.goods_number"></el-input>
    </el-form-item>
    <el-form-item label= "Catégories des produits" prop= "goods_cat">
    <el-cascader
    expand-trigger= "hover"
    v-model= "addForm.goods_cat"
    :options= "catelist"
    :props= "cateProps"
    @change= "handleChange"></el-cascader>
    </el-form-item>
    </el-tab-pane>
    <el-tab-pane label= "Paramètre des produits" name= "1">
    <el-form-item :label= "item.attr_name" v-for= "item in manyTableData" :key= "item.attr_id">
    <el-checkbox-group v-model= "item.attr_vals">
    <el-checkbox :label= "cb" v-for= "(cb, i) in item.attr_vals" :key= "i" border></el-checkbox>

  </el-checkbox-group>
    </el-form-item>
    </el-tab-pane>
    <el-tab-pane label= "Attribut des produits" name= "2">
    <el-form-item :label= "item.attr_name" v-for= "item in onlyTableData" :key= "item.attr_id">
    <el-input v-model= "item.attr_vals"></el-input>
    </el-form-item>
    </el-tab-pane>
    <el-tab-pane label= "Photo des produits" name= "3">
    <el-upload
  
  action= "uploadURL"
  :on-preview= "handlePreview"
  :on-remove= "handleRemove"
  list-type= "picture" :headers= "headerObj" :on-success= "handleSuccess">
  <el-button size="small" type="primary">Cliquer pour envoyer</el-button>
</el-upload>
    </el-tab-pane>
    <el-tab-pane label= "Contenu des produits" name= "4">
    <quill-editor v-model= "addForm.goods_introduce"></quill-editor>
    <el-button type= "primary" class= "btnAdd" @click= "add">Ajouter un produit</el-button>
    </el-tab-pane>
  </el-tabs>
</el-form>

<el-dialog
  title="Tips"
  :visible.sync= "previewVisible"
  width="50%">
  <img :src= "previewPath" alt= "" class= "previewImg"></img>
  <span slot="footer" class="dialog-footer">
    <el-button @click= "dialogVisible = false">Annuler</el-button>
    <el-button type="primary" @click= "dialogVisible = false">Confirmer</el-button>
  </span>
</el-dialog>
    </el-card>
    </div>
</template>

<script>
import _ from 'lodash'
export default {
    data(){
        return{
            activeIndex: '0',
            addForm: {
                goods_name: '',
                goods_price: 0,
                goods_weight: 0,
                goods_number: 0,
                goods_cat: [],
                pics: [],
                goods_introduce: '',
                attrs: []
            },
            addFormRules: {
                goods_name: [
                     { required: true, message: "Veuillez enter nom des produits", trigger: 'blur' },
                ],
                goods_price: [
                     { required: true, message: "Veuillez enter prix du produit", trigger: 'blur' },
                ],
                goods_weight: [
                     { required: true, message: "Veuillez enter poids du produit", trigger: 'blur' },
                ],
                goods_number: [
                     { required: true, message: "Veuillez enter la quantité du produit", trigger: 'blur' },
                ],
                goods_cat: [
                     { required: true, message: "Veuillez enter la catégorie du produit", trigger: 'blur' },
                ]
            },
            catelist: [],
            cateProps: {
                label: 'cat_name',
                value: 'cat_id',
                children:'children'
            },
            handleChange(){
                console.log(this.addForm.goods_cat)
                if(this.addForm.goods_cat.length !==3){
                    this.addForm.goods_cat = []
                }
            },
            manyTableData:[],
            onlyTableData:[],
            uploadURL: 'http://127.0.0.1:8888/api/private/v1/upload',
            headerObj: {
                Authorization: window.sessionStorage.getItem('token')
            },
            previewPath: '',
            previewVisible: false
        }

    },
    created(){
        this.getCateList()
    },
    methods: {
        async getCateList(){
            const {data:result} = await this.$http.get('categories')

            if(result.meta.status !==200) {
                return this.$message.error('Echec!')
            }

            this.catelist = result.data
            console.log(this.catelist)
        },
        beforeTableLeave(activeName, oldActiveName){
            if(oldActiveName === '0' && this.addForm.goods_cat.length !==3){
                this.$message.error('Veuillez choisir la catégorie des produits!')
                return false
            }
        },
        async tableClicked(){
            if(this.activeIndex === '1'){
                const {data:result} = await this.$http.get(`categories/${this.cateId}/attributes`, {params: {sel: 'many'}})

                if(result.meta.status !==200){
                    return this.$message.error('Echec!')
                }

                console.log(result.data)
                result.data.forEach(item => {
                    item.attr_vals =
                    item.attr_vals.length === 0 ? []:
                    item.attr_vals.split('')})
                this.manyTableData = result.data
            } else if(this.activeIndex === '2')
            {
                  const {data:result} = await this.$http.get(this.$http.get(`categories/${this.cateId}/attributes`, {params: {sel: 'only'}}))

                  if(result.meta.status !==200){
                      return this.$message.error('Echec!')
                  }

                  console.log(result.data)
                  this.onlyTableData = result.data
            }
        },
        handlePreview(file){
            console.log(file)
            this.previewPath = file.response.data.url
            this.previewVisible = true
        },
        handleRemove(file){
            const filePath = file.response.data.tmp_path
            const i = this.addForm.pics.findIndex(x => x.pic === filePath)
            this.addForm.pics.splice(i, 1)
            console.log(this.addForm)
        },
        handleSuccess(response) {
            console.log(response)

            const picInfo = {pic: response.data.tmp_path}
            this.addForm.pics.push(picInfo)
            console.log(this.addForm)
        },
        add(){
            this.$refs.addFormRef.validate(
                async valid => {
                    if(!valid){
                        return this.$message.error('Veuillez remplir les champs nécessaires du formulaire!')
                    }
                    const form = _.cloneDeep(this.addForm)
                    form.goods_cat = form.goods_cat.join(',')
                    this.manyTableData.forEach(item => {
                        const newInfo = {attr_id: item.attr_id, attr_value: item.attr_vals.join(' ')}
                        this.addForm.attrs.push(newInfo)
                    })
                    this.onlyTableData.forEach(item => {
                        const newInfo = {attr_id: item.attr_id, attr_value: item.attr_vals}
                        this.addForm.attrs.push(newInfo)
                    })
                    form.attrs = this.addForm.attrs
                    console.log(form)

                    const {data:result} = await this.$http.post('goods', form)

                    if(result.meta.status !==201){
                        return this.$message.error('Echec!')
                    }
                    this.$message.success('Succès!')
                    this.$router.push('/goods')
                })
        }
    },
    computed: {
        cateId() {
            if(this.addForm.goods_cat.length === 3){
                return this.addForm.goods_cat[2]
            }
            return null
        }
    }
}
</script>

<style lang="less" scoped>

.el-checkbox{
    margin: 0 10px 0 0 !important;
}

.previewImg{
    width: 100%;
}

.btnAdd{
    margin-top: 15px;
}
</style>
