<template>
    <div class="login_container">
    <div class="login_box">
    <div class="avatar_box">
        <img src="../assets/logo.png">
    </div>
    <el-form ref="loginFormRef":model="loginForm" :rules="loginFormRules" label-width="0px" class="login_form">
    <el-form-item prop="username">
    <el-input v-model="loginForm.username" prefix-icon="iconfont icon-user"></el-input>
    </el-form-item>
    <el-form-item prop="password">
     <el-input v-model="loginForm.password" prefix-icon="iconfont icon-3702mima" type="password"></el-input>
    </el-form-item>
    <el-form-item class="btns">
    <el-button type="primary" @click="login">Se connecter</el-button>
    <el-button type="info" @click="resetLoginForm">Réinitialiser</el-button>
    </el-form-item>
    </el-form>
    </div>
    </div>
</template>

<script>
export default {
    data(){
        return{
    loginForm:{
        username: "admin",
        password: "123456"
    },
    loginFormRules:{
      username: [
        { required: true, message: "Saisissez votre identifiant", trigger: 'blur' },
        { min: 3, max: 15, message: "Identifiant doit être de 3 à 15 caractères", trigger: 'blur' }
      ],
      password: [
        { required: true, message: "Saisissez votre mot de passe", trigger: 'blur' },
        { min: 3, max: 15, message: "Mot de passe doit être de 3 à 15 caractères", trigger: 'blur' }
      ]
    }
        };
    },
    methods: {
        resetLoginForm(){
            this.$refs.loginFormRef.resetFields();
        },
        login(){
            this.$refs.loginFormRef.validate(async valid =>{
                if (!valid) return;
                const {data:result} = await this.$http.post("login", this.loginForm);
                if(result.meta.status !==200) return this.$message.error("Echec!")
                this.$message.success("Succès");
                window.sessionStorage.setItem("token", result.data.token);
                this.$router.push("/home");
            });
        }
    }
};
</script>

<style Lang="less" scoped>
.login_container{
    background-color: yellowgreen;
    height: 100%;
}

.login_box{
    width: 550px;
    height: 400px;
    background-color: #fff;
    border-radius: 3px;
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
}

.avatar_box{
    height: 170px;
    width: 170px;
    position: absolute;
    left:47%;
    right:53%;
    transform: translate(-50%, -50%);
    img {
        width: 100%;
        height: 100%;
        border-radius: 50%;
        }
}

.login_form{
    position: absolute;
    bottom: 0;
    width: 100%;
    padding: 0 20px;
    box-sizing: border-box;
}
.btns{
    display: flex;
    justify-content: flex-end;
}
</style>
