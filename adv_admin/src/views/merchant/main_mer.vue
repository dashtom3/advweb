<template>
	<el-row class="container">
		<el-col :span="24" class="header">
			<el-col :span="10" class="logoAdmin" :class="collapsed?'logo-collapse-width':'logo-width'">
				{{collapsed?'':sysName}}
			</el-col>
			<el-col :span="10">
				<div class="tools" @click.prevent="collapse">
					<i class="fa fa-align-justify"></i>
				</div>
			</el-col>
			<el-col :span="4" class="userinfo">
				<el-dropdown trigger="hover">
					<!-- <span class="el-dropdown-link userinfo-inner"><img :src="../../images/head.png" alt="img"/> {{sysUserName}}</span> -->
					<span class="el-dropdown-link userinfo-inner"><img :src="headImg" alt="img"/> {{sysUserName}}</span>
					<el-dropdown-menu slot="dropdown">
						<el-dropdown-item>我的消息</el-dropdown-item>
						<el-dropdown-item>设置</el-dropdown-item>
						<el-dropdown-item divided @click.native="logout">退出登录</el-dropdown-item>
					</el-dropdown-menu>
				</el-dropdown>
			</el-col>
		</el-col>
		<el-col :span="24" class="main">
			<aside :class="collapsed?'menu-collapsed':'menu-expanded'">
				<!--导航菜单-->
				<el-menu :default-active="$route.path" class="el-menu-vertical-demo" @open="handleopen" @close="handleclose" @select="handleselect"
					 unique-opened router v-show="!collapsed">
					<template v-for="(item,index) in $router.options.routes" v-if="item.hidden == 'merchant'">
						<el-submenu :index="index+''" v-if="!item.leaf&&item.type.indexOf(userinfo.type)>-1">
							<template slot="title"><i :class="item.iconCls"></i>{{item.name}}</template>
							<el-menu-item v-for="child in item.children" :index="child.path" :key="child.path" v-if="!child.hidden&&child.isChildAccount.indexOf(userinfo.type) > -1">{{child.name}}</el-menu-item>
						</el-submenu>
						<el-menu-item v-if="item.leaf&&item.children.length>0&&userinfo.type!=2" :index="item.children[0].path"><i :class="item.iconCls"></i>{{item.children[0].name}}</el-menu-item>
					</template>
				</el-menu>
				<!--导航菜单-折叠后-->
				<ul class="el-menu el-menu-vertical-demo collapsed" v-show="collapsed" ref="menuCollapsed">
					<li v-for="(item,index) in $router.options.routes" v-if="!item.hidden" class="el-submenu item">
						<template v-if="!item.leaf">
							<div class="el-submenu__title" style="padding-left: 20px;" @mouseover="showMenu(index,true)" @mouseout="showMenu(index,false)"><i :class="item.iconCls"></i></div>
							<ul class="el-menu submenu" :class="'submenu-hook-'+index" @mouseover="showMenu(index,true)" @mouseout="showMenu(index,false)">
								<li v-for="child in item.children" v-if="!child.hidden" :key="child.path" class="el-menu-item" style="padding-left: 40px;" :class="$route.path==child.path?'is-active':''" @click="$router.push(child.path)">{{child.name}}</li>
							</ul>
						</template>
						<template v-else>
							<li class="el-submenu">
								<div class="el-submenu__title el-menu-item" style="padding-left: 20px;height: 56px;line-height: 56px;padding: 0 20px;" :class="$route.path==item.children[0].path?'is-active':''" @click="$router.push(item.children[0].path)"><i :class="item.iconCls"></i></div>
							</li>
						</template>
					</li>
				</ul>
			</aside>
			<section class="content-container">
				<div class="grid-content bg-purple-light">
					<el-col :span="24" class="breadcrumb-container">
						<strong class="title">{{$route.name}}</strong>
						<el-breadcrumb separator="/" class="breadcrumb-inner">
							<el-breadcrumb-item v-for="item in $route.matched" :key="item.path">
								{{ item.name }}
							</el-breadcrumb-item>
						</el-breadcrumb>
					</el-col>
					<el-col :span="24" class="content-wrapper">
						<transition name="fade" mode="out-in">
							<router-view></router-view>
						</transition>
					</el-col>
				</div>
			</section>
		</el-col>
		<!-- <audio src="../../../static/audio.wav" autoplay v-if="showAudio" > 你的游览器不支持</audio> -->
		<audio src="../../../static/audio.wav" class="audioPlay" v-model="audio"> 你的游览器不支持</audio>
	</el-row>
</template>

<script>
import global from './../global/global'
import img from '../../images/head.png'
import Vue from 'vue'
	export default {
		data() {
			return {
				sysName:'商户账号后台',
				collapsed:false,
				sysUserName: '',
				audio:null,
				userType: global.getUser().type,
				userinfo: global.getUser(),
				headImg: global.getUser().logo || img,
				webscoket: null,
				showAudio: false,
				// sysUserAvatar: '../images/head.png',
				form: {
					name: '',
					region: '',
					date1: '',
					date2: '',
					delivery: false,
					type: [],
					resource: '',
					desc: ''
				}
			}
		},
		created: function () {
			// console.log(this.$route.path)
			// console.log(this.$route)
		},
		methods: {
			onSubmit() {
				console.log('submit!');
			},
			handleopen() {
				//console.log('handleopen');
			},
			handleclose() {
				//console.log('handleclose');
			},
			handleselect: function (a, b) {
			},
			//退出登录
			logout: function () {
				var _this = this;
				this.$confirm('确认退出吗?', '提示', {
					//type: 'warning'
				}).then(() => {
					global.removeMsg()
					this.websocket.close()
					_this.$router.push('/');
				}).catch(() => {

				});
			},
			//折叠导航栏
			collapse:function(){
				this.collapsed=!this.collapsed;
			},
			showMenu(i,status){
				this.$refs.menuCollapsed.getElementsByClassName('submenu-hook-'+i)[0].style.display=status?'block':'none';
			}
		},
		mounted() {
			if (global.getUser()) {
				this.sysUserName = global.getUser().userName || '';
			}
			if (global.getToken() && this.websocket == undefined) {
				if('WebSocket' in window){
						// this.websocket = new WebSocket("ws://116.62.228.3:8080/Advertisement_proc/orderWithWs?token="+global.getToken())
						this.websocket = new WebSocket("ws://139.219.11.152:8080/Advertisement/orderWithWs?token="+global.getToken())
						// console.log(`websocket连接成功`)
						console.log(this.websocket)
						global.setWebsocket(this.websocket)

				}
				else{
						alert('Not support websocket')
				}
				var self = this
				this.websocket.onmessage = function(event){
					const msg = JSON.parse(event.data).data
					const state = msg.state == 0 ? '已经被下单' : '已经被确认'
						if (self.$route.path == '/merchant/advlist') {
							self.$notify({
								title: '成功',
								message: msg.playAdv.advertisement.name+state,
								duration: '10000',
								type: 'success'
							})
							self.$root.eventHub.$emit('shishi', msg)
						} else {
							const h = self.$createElement
							self.$notify({
								title: '成功',
								message: h('p', null, [
									h('span', null, msg.playAdv.advertisement.name+state),
									h('p', {style: 'color:#20a0ff;'} ,'点击进入广告订单页面')
								]),
								duration: '10000',
								type: 'success',
								onClick: function () {
									self.$router.push('/merchant/advlist')
								}
							})
						}
						if(msg.state == 0){
							// self.showAudio = true
							var audio = document.getElementsByClassName('audioPlay')[0];
							console.log(audio)
							audio.play();
							// setTimeout(() => {self.showAudio = false}, 2000)
						}
        }

			}
		}
	}

</script>

<style scoped lang="scss">
	@import '~scss_vars';

	.container {
		position: absolute;
		top: 0px;
		bottom: 0px;
		width: 100%;
		.header {
			height: 60px;
			line-height: 60px;
			background: $color-primary;
			color:#fff;
			.userinfo {
				text-align: right;
				padding-right: 35px;
				float: right;
				.userinfo-inner {
					cursor: pointer;
					color:#fff;
					img {
						width: 40px;
						height: 40px;
						border-radius: 20px;
						margin: 10px 0px 10px 10px;
						float: right;
					}
				}
			}
			.logoAdmin {
				//width:230px;
				height:60px;
				font-size: 22px;
				padding-left:20px;
				padding-right:20px;
				border-color: rgba(238,241,146,0.3);
				border-right-width: 1px;
				border-right-style: solid;
				margin-top: 0;
				img {
					width: 40px;
					float: left;
					margin: 10px 10px 10px 18px;
				}
				.txt {
					color:#fff;
				}
			}
			.logo-width{
				width:230px;
			}
			.logo-collapse-width{
				width:60px
			}
			.tools{
				padding: 0px 23px;
				width:14px;
				height: 60px;
				line-height: 60px;
				cursor: pointer;
			}
		}
		.main {
			display: flex;
			// background: #324057;
			position: absolute;
			top: 60px;
			bottom: 0px;
			overflow: hidden;
			aside {
				flex:0 0 230px;
				width: 230px;
				// position: absolute;
				// top: 0px;
				// bottom: 0px;
				.el-menu{
					height: 100%;
				}
				.collapsed{
					width:60px;
					.item{
						position: relative;
					}
					.submenu{
						position:absolute;
						top:0px;
						left:60px;
						z-index:99999;
						height:auto;
						display:none;
					}

				}
			}
			.menu-collapsed{
				flex:0 0 60px;
				width: 60px;
			}
			.menu-expanded{
				flex:0 0 230px;
				width: 230px;
			}
			.content-container {
				// background: #f1f2f7;
				flex:1;
				// position: absolute;
				// right: 0px;
				// top: 0px;
				// bottom: 0px;
				// left: 230px;
				overflow-y: scroll;
				padding: 20px;
				.breadcrumb-container {
					//margin-bottom: 15px;
					.title {
						width: 200px;
						float: left;
						color: #475669;
					}
					.breadcrumb-inner {
						float: right;
					}
				}
				.content-wrapper {
					background-color: #fff;
					box-sizing: border-box;
				}
			}
		}
	}
</style>
