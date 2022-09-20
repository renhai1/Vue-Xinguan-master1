<template>
  <el-container class="home-container">
<!--    导航-->
    <el-header>
      <div style="margin-left:20px;">
        <img
            src="../assets/ilogo.png"
            width="50px;"
            style="margin-left:-10px; border-radius: 50%;"
            height="100%;"
            alt
            srcset
          />
           <h1>抗疫物资溯源管控平台</h1>
      </div>
      <el-dropdown>
        <div class="block">
          <el-avatar  :size="50" :src="this.userInfo.avatar" style="cursor: pointer;"></el-avatar>
        </div>
        <el-dropdown-menu slot="dropdown" trigger="click">
           <el-dropdown-item>
             <span type="danger"  @click="toWelcome"><span class="el-icon-house"></span> &nbsp;系统首页</span>
          </el-dropdown-item>

          <el-dropdown-item>

            <span type="danger" @click="logout"><span class="el-icon-switch-button"></span> &nbsp;退出登入</span>

          </el-dropdown-item>
        </el-dropdown-menu>
      </el-dropdown>
    </el-header>
    <!--主体-->
    <el-container style="height: 500px;">
      <!--菜单-->
      <el-aside :width="isOpen==true?'64px':'200px'">
        <div class="toggle-btn" @click="toggleMenu">|||</div>
        <el-menu
          class="el-menu-vertical-demo"
          :collapse="isOpen"
          :router="true"
          :default-active="activePath"
          background-color="rgba(0,0,0,0)"
          :collapse-transition="false"
          text-color="#303133"
          unique-opened
        >
          <MenuTree  :menuList="this.menuList"></MenuTree>
        </el-menu>
      </el-aside>

      <!--右边主体-->
      <el-main v-loading="loading">

        <router-view></router-view>

      </el-main>
    </el-container>
  </el-container>
</template>

<script>
import MenuTree from "./MenuTree.vue"; //引进菜单模板

export default {
  data() {
    return {
      loading: true,
      activePath: "", //激活的路径
      isOpen: false,
      menuList: {},
      userInfo: {},

    };
  },
  components: {
    MenuTree
  },
  methods: {
    /**
     *
     * 退出登入
     */
    async logout() {
      var res = await this.$confirm("此操作将退出系统, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      }).catch(() => {
        this.$message({
          type: "info",
          message: "已取消退出登入"
        });
      });
      if (res == "confirm") {
        window.localStorage.clear();
        this.$router.push("/login");
      }
    },
    /**
     * 去系统首页
     */
    toWelcome(){
      this.$router.push("/welcome");
    },
    /**
      加载菜单数据
     */
    async getMenuList() {
      const { data: res } = await this.$http.get("user/findMenu");
      if (res.code !== 200)
        return this.$message.error("获取菜单失败:" + res.msg);
       this.menuList = res.data;
    },
    /**
      获取用户信息
     */
    async getUserInfo() {
      const { data: res } = await this.$http.get("user/info");
      if (res.code !== 200) {
        return this.$message.error("获取用户信息失败:" + res.msg);
      } else {
        this.userInfo = res.data;
        //保存用户
        this.$store.commit("setUserInfo", res.data);
      }
    },
    /**
     * 菜单伸缩
     */
    toggleMenu() {
      this.isOpen = !this.isOpen;
    },

    /**
     * 点击交流
     */
    getContact(){
      const w = window.open('about:blank');
      w.location.href = 'https://www.zykcoderman.xyz/';
    }

  },
  mounted() {
    this.getUserInfo();
  },
  created() {
    this.getMenuList();
    this.activePath = window.sessionStorage.getItem("activePath");
    // if(window.sessionStorage.getItem("activePath")==null){
    //   this.activePath='/welcome';
    // }
    setTimeout(() => {
      this.loading = false;
    }, 500);
  }
};
</script>

<style>
/* 为对应的路由跳转时设置动画效果 */

.el-header {
  background-color: #fff;
  box-shadow: 0 15px 37px rgb(0 0 0);
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 19px;

  padding-left: 0px;
}

.el-header div{
  display: flex;
  align-items: center;
}
.el-header div img{
  margin-right: 20px;
}
.el-header div h1{
  font-weight: bolder;
  color: #1f2935;
  font-family: 'Courier New', Courier, monospace;
}
.el-header 
.el-aside {
  background-color:#001529
}
.el-main {
  background-color: #eaedf1;
}
.home-container {
  width: 100%;
  height: 100% !important;
}
.toggle-btn {
  background-color: rgb(48,65,86) !important;
  font-size: 10px;
  line-height: 24px;
  color: #fff;
  text-align: center;
  letter-spacing: 0.2em;
  cursor: pointer;
}

</style>
