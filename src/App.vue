<template>
  <v-app>
    <v-app-bar app color="primary" dark v-if="isHome">
      <v-app-bar-nav-icon @click.stop="drawer=!drawer"></v-app-bar-nav-icon>
      <v-toolbar-title>{{appName}}</v-toolbar-title>

      <v-spacer></v-spacer>

      <v-btn icon @click="setDialogComponent('cart')">
        <v-badge v-if="countCart<=0">
            <v-icon>mdi-cart</v-icon>
        </v-badge>
        <v-badge color="orange" overlap v-else>
          <template v-slot:badge>
            <span>{{countCart}}</span>
          </template>
           <v-icon>mdi-cart</v-icon>
        </v-badge>
      </v-btn>
      <v-text-field slot="extension" hide-details 
      append-icon="mdi-microphone" flat label="Search" solo-inverted 
      prepend-inner-icon="mdi-magnify" @click="setDialogComponent('search')">
      </v-text-field>
    </v-app-bar>

    <v-app-bar app color="primary" dark="" v-else>
      <v-btn icon @click.stop="$router.go(-1)">
        <v-icon>mdi-arrow-left-circle</v-icon>
      </v-btn>
      <v-spacer></v-spacer>

      <v-btn icon @click="setDialogComponent('cart')">
        <v-badge v-if="countCart<=0">
            <v-icon>mdi-cart</v-icon>
        </v-badge>
        <v-badge color="orange" overlap v-else>
          <template v-slot:badge>
            <span>{{countCart}}</span>
          </template>
           <v-icon>mdi-cart</v-icon>
        </v-badge>
      </v-btn>
    </v-app-bar>

    <v-card>
      <v-navigation-drawer app v-model="drawer">
        <v-list>
          <v-list-item v-if="!guest">
            <v-list-item-avatar>
              <v-img :src="getImage('/users/',user.avatar)"></v-img>
            </v-list-item-avatar>
            <v-list-item-content>{{user.name}}</v-list-item-content>
          </v-list-item>
          
          <div class="pa-2" v-if="guest">
            <v-btn block color="primary" class="mb-1" @click="setDialogComponent('login')"><v-icon left>mdi-lock</v-icon>Login</v-btn>
            <v-btn block color="success" @click="setDialogComponent('register')"><v-icon left>mdi-account</v-icon>Register</v-btn>
          </div>

          <v-divider></v-divider>

          <v-list shaped>
            <template v-for="(item,index) in menus">
              <!-- <v-list-item v-for="(item,index) in menus" :key="`menu-`+index" :to="item.route"> -->
              <v-list-item :key="`menu-`+index" :to="item.route" v-if="!item.auth || (item.auth && !guest)">
                <v-list-item-icon>  
                  <v-icon left>{{item.icon}}</v-icon>
                </v-list-item-icon>
                <v-list-item-content>
                  <v-list-item-title>{{item.title}}</v-list-item-title>
                </v-list-item-content>
              </v-list-item>
            </template>
          </v-list>

          
        </v-list>
        
        <template v-slot:append v-if="!guest">
          <div class="pa-2">
            <v-btn block color="red" dark @click="logout"><v-icon left>mdi-lock</v-icon>Logout</v-btn>
          </div>
        </template>
      </v-navigation-drawer>
    </v-card>

    <alert />
    
    <keep-alive>
      <v-dialog v-model="dialog" fullscreen="" hide-overlay="" transition="dialog-bottom-transition">
        <!-- <search @closed="closeDialog" /> -->
        <component :is="currentComponent" @closed="setDialogStatus"></component>
      </v-dialog>
    </keep-alive>

    <v-content>
      <v-container fluid>
        <!-- <HelloWorld /> -->
        <v-slide-y-transition>
          <router-view></router-view>
        </v-slide-y-transition>
      </v-container>  
    </v-content>

    <v-card>
      <v-footer app absolute>
        <v-card-text class="text-center">&copy; {{new Date().getFullYear()}} - <strong>{{appName}}</strong></v-card-text>
    </v-footer>
    </v-card>
  </v-app>
</template>

<script>
// import HelloWorld from './components/HelloWorld';
import {mapGetters, mapActions} from 'vuex'
export default {
  name: 'App',
  components: {
    // HelloWorld,
    Alert:()=>import('@/components/Alert.vue'),
    Search:()=>import('@/components/Search.vue'),
    Login:()=>import('@/components/Login.vue'),
    Register:()=>import('@/components/Register.vue'),
    Cart:()=>import('@/components/cart.vue')
  },

  // mounted(){
  //   console.log(process.env)
  // },

  data: () => ({
    // title: process.env.VUE_APP_NAME,
    drawer:false,
    menus:[
      {title:'Home',icon:'mdi-home',route:'/'},
      {title:'Profile',icon:'mdi-account',route:'/profile',auth:true},
      // {title:'About',icon:'mdi-account',route:'/about'},
      {title:'My Orders',icon:'mdi-shopping',route:'/orders',auth:true},
    ],
    // guest:true,
    // dialog:false,
  }),
  computed:{
    isHome(){
      return (this.$route.path==='/')
    },
    ...mapGetters({
      countCart:'cart/count',
      guest:'auth/guest',
      user:'auth/user',
      dialogStatus:'dialog/status',
      currentComponent:'dialog/component'
    }),
    dialog:{
      get(){
        return this.dialogStatus
      },
      set(value){
        this.setDialogStatus(value)
      }
    }
  },
  methods:{
    // closeDialog(value){
    //   this.dialog=value
    // },
    ...mapActions({
      setDialogStatus:'dialog/setStatus',
      setDialogComponent:'dialog/setComponent',
      setAuth:'auth/set',
      setProfile:'auth/setProfile',
      setAlert:'alert/set',
    }),
    logout(){
      let config={
        headers:{
          'authorization': 'Bearer ' + this.user.api_token
        },
      }
      this.axios.post('/logout',{},config)
      .then((response)=>{
        console.log(response)
        this.setAuth({})
        this.setProfile({})
        this.setAlert({
          status:true,
          color:'success',
          text:'Logout successfully',
        })
      })
      .catch((error)=>{
        let {data} = error.response
        console.log(data)
        this.setAlert({
          status:true,
          color:'error',
          text:data.message,
        })
      })
    },
  },
  // mounted(){
  //   window.Echo.channel('test-event')
  //   .listen('ExampleEvent', (e) => {
  //       console.log(e);
  //   });
    // console.log(this.user.api_token)
    // // console.log(this.$auth.$storage._state['api_token.local'])

    // window.Echo.connector.options.auth.headers['Authorization'] = 'Bearer ' + this.user.api_token
    // // this.$auth.$storage._state['_token.local']
    // // window.Echo.private('App.User.' + this.$auth.user.id).notification(
    // window.Echo.private('App.User.' + this.user.id).notification(
    //   notif => {
    //     console.log(notif)
    //   }
    // )
  // },
};
</script>
