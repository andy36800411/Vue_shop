<template>
    <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
    <el-breadcrumb-item :to="{ path: '/home' }">Accueil</el-breadcrumb-item>
    <el-breadcrumb-item>Gestion des droits</el-breadcrumb-item>
    <el-breadcrumb-item>Liste des rôle</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
        <el-row>
         <el-col>
            <el-button type= "primary">Ajouter le rôle</el-button>
            </el-col>   
        </el-row>
        <el-table :data= "rolelist" border stripe>
        <el-table-column type= "expand">
        <template slot-scope= "scope">
            <el-row :class= "['bdbottom', i1 === 0 ? 'bdtop' : '', 'vcenter']" v-for="(item1, i1) in scope.row.children" :key= "item1.id">
                <el-col :span= "5">
                    <el-tag closable @close= "removeRightById(scope.row, item1.id)">{{item1.authName}}</el-tag>
                    <i class="el-icon-caret-right"></i>
                </el-col>
                <el-col :span= "19">
                    <el-row :class= "[i2 === 0 ? '' : 'bdtop', 'vcenter']" v-for= "(item2, i2) in item1.children" :key= "item2.id">
                        <el-col :span= "6">
                            <el-tag type= "success" closable @close= "removeRightById(scope.row, item2.id)">{{item2.authName}}</el-tag>
                            <i class="el-icon-caret-right"></i>
                        </el-col>
                        <el-col :span ="18">
                        <el-tag type= "warning" v-for= "(item3, i3) in item2.children" :key= "item3.id" closable @close= "removeRightById(scope.row, item3.id)">{{item3.authName}}</el-tag>
                        </el-col>
                    </el-row>
                </el-col>
            </el-row>
            <pre>
             {{scope.row}}
            </pre>
        </template>
        </el-table-column>
        <el-table-column type= "index"></el-table-column>
        <el-table-column label= "Nom des Rôles" prop= "roleName"></el-table-column>
        <el-table-column label= "Description des Rôles" prop= "roleDesc"></el-table-column>
        <el-table-column label= "Opération" width= "420px">
        <template slot-scope= "scope">
        <el-button size= "mini" type="primary" icon="el-icon-edit">Édition</el-button>
        <el-button size= "mini" type="danger" icon="el-icon-delete">Suppression</el-button>
        <el-button size= "mini" type="warning" icon="el-icon-setting" @click= "showSetRightDialog(scope.row)">Attribution des droits</el-button>
        </template>
        </el-table-column>
        </el-table>
    </el-card>

    <el-dialog
  title="Attribution des droits"
  :visible.sync= "showSetRightDialogVisible"
  width="50%" @close= "setRightDialogClosed">
  <el-tree :data= "rightslist" :props= "treeProps" show-checkbox node-key= "id" default-expand-all :default-checked-keys= "defkeys" ref= "treeRef"></el-tree>
  <span slot="footer" class= "dialog-footer">
    <el-button @click= "showSetRightDialogVisible = false">Annuler</el-button>
    <el-button type="primary" @click= "allotRights">Confirmer</el-button>
  </span>
</el-dialog>
    </div>
</template>

<script>
export default {
    data() {
        return {
            rolelist: [],
            showSetRightDialogVisible: false,
            rightslist: [],
            treeProps: {
                label: 'authName',
                children: 'children'
            },
            defkeys: [],
            roleId: ''
        }
    },
    created() {
        this.getRolesList()
    },
    methods: {
        async getRolesList() {
            const {data:result} = await this.$http.get('roles')

            if(result.meta.status !==200) {
                return this.$message.error("Échec de l'obtention d'une liste de rôle")
            }

            this.rolelist = result.data

            console.log(this.rolelist)
        },
        async removeRightById (role, rightId) {
        const confirmResult = await this.$confirm('Ceci effacera le fichier. Continuer?', 'Warning', {
          confirmButtonText: 'OK',
          cancelButtonText: 'Annuler',
          type: 'warning'
        }).catch(err => err)

        if(confirmResult !== 'confirm') {
            return this.$message.info('Arrêter la suppression')
        }
        const {data:result} = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)

        if(result.meta.status !==200) {
            return this.$message.error('Échec de la suppression du droit !')
        }

        role.children = result.data
        },
        async showSetRightDialog(role) {
            this.roleId = role.id
            const {data:result} = await this.$http.get('rights/tree')

            if(result.meta.status !==200) {
                return this.$message.errorˇ("Échec de l'obtention des données")
            }

            this.rightslist = result.data
            console.log(this.rightslist)

            this.getLeafkeys(role, this.defkeys)
            this.showSetRightDialogVisible = true
        },
        getLeafkeys(node, arr) {
            if(!node.children){
                return arr.push(node.id)
            }

            node.children.forEach(item => 
            this.getLeafkeys(item, arr))
        },
        setRightDialogClosed() {
            this.defkeys = []
        },
        async allotRights() {
            const keys = [
                ...this.$refs.treeRef.getCheckedKeys(),
                ...this.$refs.treeRef.getHalfCheckedKeys()
            ]

            const idStr = keys.join(',')

            const {data:result} = await this.$http.post(`roles/${this.roleId}/rights`, {rids: idStr})

            if(result.meta.status !==200) {
                return this.$message.error("Échec de l'attribution des droits")
            }

            this.$message.success("Succès de l'attribution des droits")
            this.getRolesList()
            this.setRightDialogVisible = false
        }
    }
}
</script>

<style lang="less" scoped>

.el-tag{
    margin:7px;
}

.bdtop{
    border-top: 1px solid #eee;
}

.bdbottom {
    border-bottom: 1px solid #eee; 
}

.vcenter {
    display: flex;
    align-items: center;
}

</style>
