<template>
    <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
    <el-breadcrumb-item :to="{ path: '/home' }">Accueil</el-breadcrumb-item>
    <el-breadcrumb-item>Gestion promotions</el-breadcrumb-item>
    <el-breadcrumb-item>Liste promotions</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
    
    <el-row :gutter="20">
    <el-col :span= "8">
    <el-input placeholder="Entrez quelque chose" v-model= "queryInfo.query" clearable @clear= "getUserList">
    <el-button slot="append" icon="el-icon-search" @click= "getUserList"></el-button>
    </el-input>
    </el-col>
    <el-col :span= "3"></el-col>
    <el-button type="primary" @click= "addDialogVisible = true">Ajouter un utilisateur</el-button>
    </el-row>
    <el-table :data = "userlist" border stripe>
    <el-table-column type="index"></el-table-column>
    <el-table-column label="Nom" prop="username"></el-table-column>
    <el-table-column label="Email" prop="email"></el-table-column>
    <el-table-column label="Téléphone" prop="mobile"></el-table-column>
    <el-table-column label="Rôle" prop="role_name"></el-table-column>
    <el-table-column label="Statut">
     <template slot-scope="scope">
        <el-switch v-model= "scope.row.mg_state" @change= "userStateChanged(scope.row)">
        </el-switch>
     </template>
    </el-table-column>
    <el-table-column label="Opération" width="180px">
        <template slot-scope="scope">
          <el-button type="primary" icon="el-icon-edit" size="mini" @click= "showEditDialog(scope.row.id)"></el-button>
          <el-button type="danger" icon="el-icon-delete" size="mini" @click= "removeUserById(scope.row.id)"></el-button>
        <el-tooltip  effect="dark" content="Attribution des rôles" placement="top" :enterable = "false">
        <el-button type="warning" icon="el-icon-setting" size="mini"></el-button>  
        </el-tooltip> 
        </template>
    </el-table-column>
    </el-table>
    <el-pagination
      @size-change= "handleSizeChange"
      @current-change= "handleCurrentChange"
      :current-page= "queryInfo.pagenum"
      :page-sizes="[1, 2, 5, 10]"
      :page-size= "queryInfo.pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total= "total">
    </el-pagination>
    </el-card>
    <el-dialog
    title="Ajouter un utilisateur"
    :visible.sync= "addDialogVisible"
    width="50%" @close= "addDialogClosed('addFormRef')">
    <el-form :model= "addForm" :rules= "addFormRules" ref="addFormRef" label-width="140px">
  <el-form-item label="Nom de l'utilisateur" prop="username">
    <el-input v-model= "addForm.username"></el-input>
  </el-form-item>
  <el-form-item label="Mot de passe" prop="password">
    <el-input v-model= "addForm.password"></el-input>
  </el-form-item>
  <el-form-item label="L'émail" prop="email">
    <el-input v-model= "addForm.email"></el-input>
  </el-form-item>
  <el-form-item label="Téléphone" prop="mobile">
    <el-input v-model= "addForm.mobile"></el-input>
  </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
    <el-button @click= "addDialogVisible = false">Supprimer</el-button>
    <el-button type="primary" @click= "addUser">Valider</el-button>
    </span>
    </el-dialog>
    <el-dialog
  title="Modifier un utilisateur"
  :visible.sync= "editDialogVisible"
  width="50%" @close= "editDialogClosed">
  <el-form :model= "editForm" :rules= "editFormRules" ref="editFormRef" label-width="140px">
  <el-form-item label="Nom de l'utilisateur">
    <el-input v-model= "editForm.username" disabled></el-input>
  </el-form-item>
   <el-form-item label="Email" prop= "email">
    <el-input v-model= "editForm.email"></el-input>
  </el-form-item>
   <el-form-item label="Téléphone" prop= "mobile">
    <el-input v-model= "editForm.mobile"></el-input>
  </el-form-item>
  </el-form>
  <span slot="footer" class="dialog-footer">
    <el-button @click= "editDialogVisible = false">Supprimer</el-button>
    <el-button type="primary" @click= "editDialogVisible = false">Valider</el-button>
  </span>
</el-dialog>
        </div>
    </template>

<script>
export default {
    data() {
        var checkEmail = (rule, value, cb) => {
            const regEmail = /^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+(\.[a-zA-Z0-9_-])+/
            if (regEmail.test(value)) {
                return cb()
            }
            cb(new Error('Veuillez entrer email correct'))
        }
        var checkMobile = (rule, value, cb) => {
            const regMobile = /^(0|86|17951)?(13[0-9]|15[012356789]|17[678]|18[0-9]|14[57])[0-9]{8}$/
            if(regMobile.test(value)) {
                return cb()
            }
            cb(new Error('Veuillez entrer téléphone correct'))
        }
        return {
        queryInfo: {
            query: '',
            pagenum: 1,
            pagesize: 2
            },
            userlist: [],
            total: 0, 
            addDialogVisible: false,
            addForm: {
                username:'',
                password:'',
                email:'',
                mobile:''
            },
            addFormRules: {
                username: [
                { required: true, message: "Veuillez enter votre nom de l'utilisateur", trigger: 'blur' },
                { min: 3, max: 10, message: "Nom de l'utilisateur doit être de 3 à 10 caractères", trigger: 'blur' }
                ],
                password: [
                { required: true, message: "Veuillez enter votre mot de passe", trigger: 'blur' },
                { min: 6, max: 15, message: "Mot de passe doit être de 6 à 15 caractères", trigger: 'blur' }
                ],
                email:  [
                { required: true, message: "Veuillez enter votre email", trigger: 'blur' },
                { validator: checkEmail, trigger: 'blur' }
                
                ],
                mobile: [
                { required: true, message: "Veuillez enter votre téléphone", trigger: 'blur' },
                { validator: checkMobile, trigger: 'blur' }
                ], 
            },
                editDialogVisible: false,
                editForm: {},
                editFormRules: {
                email:  [
                { required: true, message: "Veuillez enter votre email", trigger: 'blur' },
                { validator: checkEmail, trigger: 'blur' }
                ],
                 mobile: [
                { required: true, message: "Veuillez enter votre téléphone", trigger: 'blur' },
                { validator: checkMobile, trigger: 'blur' }
                ], 
                }
        }
    },
    created() {
        this.getUserList()
    },
    methods: {
        async getUserList() {
        const { data: result } = await this.$http.get('users', { 
            params: this.queryInfo
        })
        if(result.meta.status !== 200) {
           return this.$message.error('Échec de la récupération de la liste des utilisateurs!')
        }
        this.userlist = result.data.users
        this.total = result.data.total
        console.log(result)
    },
    handleSizeChange(newSize){
        console.log(newSize)
        this.queryInfo.pagesize = newSize
        this.getUserList()
    },
    handleCurrentChange(newPage){
        console.log(newPage)
        this.queryInfo.pagesize = newPage
        this.getUserList()
    },
    async userStateChanged(userinfo){
        console.log(userinfo)
        const {data:result} = await this.$http.put(`users/${userinfo.id}/state/${userinfo.mg_state}`)
        if(result.meta.status !== 200){
            userinfo.mg_state = !userinfo.mg_state
            return this.$message.error("Échec de la mise à jour du statut de l'utilisateur!")
        }
        this.$message.success("Mise à jour du statut de l'utilisateur réussie!")
    },
    addDialogClosed() {
        this.$refs.addFormRef.resetFields()
    },
    addUser() {
        this.$refs.addFormRef.validate(async valid => 
        {
            if ( !valid ) return
            const { data:result } = await this.$http.post('users', this.addForm)
            if( result.meta.status !== 201 ) {
                this.$message.error('Échec de la mise à jour du statut de utilisateur!')
            }
            this.$message.success('Mise à jour du statut de utilisateur réussie!')
            this.addDialogVisible = false
            this.getUserList()
        })
    },
    async showEditDialog(id) {
           this.editDialogVisible = true
           const { data:result } = await this.$http.get('users/' + id)
           if(result.meta.status !== 200) {
               return this.$message.error('Échec!')
           }
           this.editForm = result.data
           this.editDialogVisible = true
    },
    editDialogClosed() {
        this.$refs.editFormRef.clearValidate()
    },
    editUserInfo() {
        this.$refs.editFormRef.validate(async valid => {
            if (!valid) return
            const { data: result } = await this.$http.put(
                'users/' + this.editForm.id, 
                {
                email: this.editForm.email, 
                mobile: this.editForm.mobile
                }
        )

        if(result.meta.status !== 200) {
            return this.$message.error('Échec de la mise à jour des informations sur utilisateur!')
        }
        this.editDialogVisible = false
        this.getUserList()
        this.$message.success('Mise à jour des informations sur utilisateur réussie!')
    })
    },
    async removeUserById(id){
        const confirmResult = await this.$confirm('Ceci effacera le utilisateur. Continuer?', 'Alerte', {
          confirmButtonText: 'OK',
          cancelButtonText: 'Annuler',
          type: 'warning'
        }).catch(err => err)
        if(confirmResult !== 'confirm') {
            return this.$message.info('Annulé')
        }
        const {data:result} = await this.$http.delete('users/' + id)
        if(result.meta.status !==200){
            return this.$message.error('Échec de la suppression de utilisateur')
        }
        this.$message.success('Succès de la suppression de utilisateur')
        this.getUserList()
    }
}
}
</script>

<style Lang="less" scoped>
</style>
