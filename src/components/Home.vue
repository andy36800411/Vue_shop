<template>
<el-container class="home-container">
  <el-header>
  <div>
    <img src="../assets/Vins.png" alt="">
    <span>Commerce du vin</span>
  </div>
   <el-button type="info" @click="logout">Quitter</el-button>
  </el-header>
  <el-container>
    <el-aside :width = "isCollapse ? '64px' : '200px'">
      <div class="toggle-button" @click = "toggleCollapse">|||</div>
      <el-menu background-color="#40E0D0" text-color="#000080" active-text-color="#ffd04b" unique-opened :collapse = "isCollapse"
      :collapse-transition = "false" router :default-active= "activePath">
      <el-submenu :index = "item.id + ''" v-for = "item in menulist" :key = "item.id">
        <template slot="title">
          <i :class = "iconsObj[item.id]"></i>
          <span>{{ item . authName }}</span>
        </template>

       <el-menu-item :index = "'/' + subItem.path" v-for = "subItem in item.children" 
       :key = "subItem.id" @click= "saveNavState('/' + subItem.path)">
        <template slot="title">
          <i class="el-icon-menu"></i>
          <span>{{ subItem . authName }}</span>
        </template>
        </el-menu-item>
      </el-submenu>
    </el-menu>
    </el-aside>
    <el-main>
    <router-view></router-view>
    </el-main>
  </el-container>
</el-container>
</template>

<script>
export default {
    data() {
        return {
        menulist: [],
        iconsObj: {
          125: 'iconfont icon-user',
          103: 'iconfont icon-tijikongjian',
          101: 'iconfont icon-shangpin',
          102: 'iconfont icon-danju',
          145: 'iconfont icon-baobiao',
        },
        isCollapse: false,
        activePath: ''
        }
    },
    created() {
        this.getMenulist()
        this.activePath = window.sessionStorage.getItem('activePath')
    },
    methods: {
        logout() {
            window.sessionStorage.clear()
            this.$router.push("/login")
        },
        async getMenulist() {
            const {data:result} = await this.$http.get('menus')
            if(result.meta.status !== 200) return this.$message.error(result.meta.msg)
            this.menulist = result.data
            console.log(result);
        },
        toggleCollapse(){
          this.isCollapse = ! this.isCollapse
          },
          saveNavState(activePath) {
          window.sessionStorage.setItem ('activePath', activePath)
          this.activePath = activePath
          }
    }
}
</script>

<style Lang="less" scoped>

.home-container{
    height: 100%;
}

.el-header {
    background-color: #FFF0F5;
    display: flex;
    justify-content: space-between;
    padding-left: 0;
    align-items: center;
    color:#4169E1;
    font-size: 20px;
}

.el-header > div {
    display: flex;
    align-items: center;
}

.el-header > div > span{
    margin-left: 15px;
}

.el-aside {
    background-color: #40E0D0;
}

.el-menu {
    border-right: none;
}

.el-main {
    background-color: #eaedf1;
}

.iconfont{
  margin-right: 10px;
}

.toggle-button{
  background-color: #4A5064;
  font-size: 10px;
  line-height: 24px;
  color: #fff;
  text-align: center;
  letter-spacing: 0.2em;
  cursor: pointer;
}

</style>
