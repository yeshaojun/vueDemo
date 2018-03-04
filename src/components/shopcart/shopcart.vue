<template>
	<div class="shopcart">
		<div class="content" @click="toggleList">
			<div class="content-left">
				<div class="logo-wrapper">
					<div class="logo" :class="{'heightlight':totalCount>0}">
						<span class="icon-shopping_cart" :class="{'heightlight':totalCount>0}"></span>
					</div>
					<div class="number" v-show="totalCount>0">
						{{totalCount}}
					</div>
				</div>
				<div class="price" :class="{'heightlight':totalCount>0}">
					¥{{totalPrice}}
				</div>
				<div class="desc">
					另需配送费¥{{deliveryPrice}}
				</div>
			</div>
			<div class="content-right" >
				<div class="pay" :class="{'enough':payDesc=='去结算'}">
					{{payDesc}}
				</div>
			</div>
		</div>
		<div class="ball-container">
		  <transition name="drop" v-on:before-enter="beforeEnter" v-on:enter="enter" v-on:after-enter="afterEnter" v-for="(ball,index) in balls">
		    <div class="ball" v-show="ball.show" >
		      <div class="inner inner-hook"></div>
		    </div>
		  </transition>
		</div>
		<transition name="fold">
			<div class="shopcart-list" v-show="listShow">
				<div class="list-header">
					<h1 class="title">购物车</h1>
					<span class="empty" @click="setEmpty()">清空</span>
				</div>
				<div class="list-content" ref="listContent">
					<ul>
						<li class="food" v-for="food in selectFoods">
							<span class="name">{{food.name}}</span>
							<div class="price">
								<span>¥{{food.price*food.count}}</span>
							</div>
							<div class="cartcontrol-wrapper">
								<cartcontrol :food="food"></cartcontrol>
							</div>
						</li>
					</ul>
				</div>
			</div>
		</transition>
	</div>

</template>

<script type="text/ecmascript-6">
import BScroll from 'better-scroll';
import cartcontrol from '../cartcontroll/cartcontroll.vue';

	export default{
		data(){
			return {
				balls:[
					{show:false},
					{show:false},
					{show:false},
					{show:false},
					{show:false}
				],
				dropBalls:[],
				fold:true
			}
		},
		props:{
			deliveryPrice:{
				type:Number
			},
			minPrice:{
				type:Number
			},
			selectFoods:{
				type:Array,
				default(){
					return []
				}
			}
		},
		computed:{
			totalPrice(){
				let total=0;
				this.selectFoods.forEach((food)=>{
					total+=food.price*food.count;
				})
				return total;
			},
			totalCount(){
				let count=0;
				this.selectFoods.forEach((food)=>{
					count+=food.count;
				})
				return count;
			},
			payDesc(){
				if(this.totalPrice===0){
					return `¥${this.minPrice}元起送`;
				}else if(this.totalPrice<this.minPrice){
					let money=this.minPrice-this.totalPrice;
					return `还差¥${money}元起送`;
				}else{
					return '去结算';
				}
			},
			listShow(){
				if(!this.totalCount>0){
					this.fold=true;
					return false;
				}else{
					let show=this.fold;

					if(show){
						this.$nextTick(()=>{
							if(!this.foodlistScroll){
								this.foodlistScroll=new BScroll(this.$refs.listContent,{
									click:true
								});
							}else{
								this.foodlistScroll.refresh();
							}
	
						})
					}
					return show;
				}
			}
		},
		methods:{
			drop(el){	
				for (let i = 0, l = this.balls.length; i < l; i++) {
				  let ball = this.balls[i]
				  if (!ball.show) {
				    ball.show = true
				    ball.el = el
				    this.dropBalls.push(ball)
				    return
				  }
				}		
			},
			setEmpty() {
			  this.selectFoods.forEach((food) => {
			    food.count = 0
			  })
			},
			hideBackdrop() {
			  this.listShow = false
			},
			beforeEnter(el) {
			  let count = this.balls.length
			  while (count--) {
			    let ball = this.balls[count]
			    if (ball.show) {
			      let rect = ball.el.getBoundingClientRect()
			      let x = rect.left - 32;
			      let y = -(window.innerHeight - rect.top - 22);
			      el.style.display = 'shopcart.vue';
			      el.style.webkitTransform = `translate3d(0,${y}px,0)`;
			      el.style.transform = `translate3d(0,${y}px,0)`;
			      let inner = el.querySelector('.inner-hook');
			      inner.style.webkitTransform = `translate3d(${x}px,0,0)`;
			      inner.style.transform = `translate3d(${x}px,0,0)`;
			    }
			  }
			},
			enter(el) {
				console.log(el);
			  el.offsetHeight // 触发浏览器重绘，offsetWidth、offsetTop等方法都可以触发
			  this.$nextTick(() => {
			    el.style.webkitTransform = 'translate3d(0,0,0)';
			    el.style.transform = 'translate3d(0,0,0)';
			    let inner = el.querySelector('.inner-hook');
			    inner.style.webkitTransform = 'translate3d(0,0,0)';
			    inner.style.transform = 'translate3d(0,0,0)';
			  })
			},
			afterEnter(el) {
			  let ball = this.dropBalls.shift();
			  if (ball) {
			    ball.show = false;
			    el.style.display = 'none';
			  }
			},
			toggleList(){
				if(!this.totalCount){
					return; 
				}
				this.fold=!this.fold;
			}
		},
		created() {
		    this.$root.eventHub.$on('cart.add', this.drop);
		},
		components:{
			cartcontrol
		}
	}
</script>
<style lang="less">
	@import '../../common/less/mixin.less';
	.shopcart{
		position:fixed;
		left:0;
		bottom: 0;
		z-index: 50;
		width: 100%;
		height:48px;
		.content{
			display: flex;
			background-color: #141d27;
			font-size: 0;
			.content-left{
				flex: 1;
				.logo-wrapper{
					display: inline-block;
					position: relative;
					top:-10px;
					margin:0 12px;
					padding: 6px;
					width:56px;
					height: 56px;
					box-sizing: border-box;
					vertical-align: top;
					border-radius:50%;
					background: #141d27;
					.logo{
						width: 100%;
						height: 100%;
						background-color: #2b343c;
						text-align: center;
						border-radius: 50%;
						.icon-shopping_cart{
							font-size: 24px;
							color:#80858a;
							line-height: 44px;
							&.heightlight{
								background: rgb(0,160,220);
								color: #fff;
							}
						}
						&.heightlight{
							background: rgb(0,160,220);
							color: #fff;
						}
					}
					.number{
						position: absolute;
						top:0;
						right:0;
						width:24px;
						height: 16px;
						line-height: 16px;
						text-align: center;
						border-radius: 16px;
						font-size: 9px;
						font-weight: 700;
						color: #fff;
						background: rgb(240,20,20);
						box-shadow: 0 4px 8px 0 rgba(0,0,0,0.4);
					}
				}
				.price{
					display: inline-block;
					vertical-align: top;
					line-height: 24px;
					margin-top: 12px;
					box-sizing: border-box;
					padding-right:12px;
					border-right: 1px solid rgba(255,255,255,0.1);
					font-size: 16px;
					font-weight: 700;
					color:rgba(255,255,255,.4);
					&.heightlight{
						color: #fff;		
					}
				}
				.desc{
					display: inline-block;
					vertical-align: top;
					line-height: 24px;
					margin: 12px 0 0 12px;
					color:rgba(255,255,255,0.4); 
					font-size: 10px;

				}
			}
			.content-right{
				flex:0 0 105px;
				width: 105px;
				.pay{
					height: 48px;
					line-height: 48px;
					text-align: center;
					font-size: 12px;
					color: rgba(255,255,255,.4);
					font-weight: 700;
					background: #2b333b;				
					&.enough{
						background: #00b43c;
						color: #fff;	
					}
				}
			}
		}
		.ball-container{
			.ball{
				position: fixed;
				left:32px;
				bottom: 22px;
				z-index: 200;
				&.drop-enter, &.drop-enter-active{
					transition: all 0.4s linear;
					.inner{
						width: 16px;
						height: 16px;
						border-radius: 50%;
						background:rgb(0,160,220);
						transition: all 0.4s linear;
					}
				}
			}
		}
		.shopcart-list{
			position: absolute;
			top:0;
			left:0;
			z-index:-1;
			width: 100%; 
			transform:translate3d(0,-100%,0);
			&.fold-enter-active, &.fold-leave-active{
			  transition: all 0.5s;
			}
			&.fold-enter, &.fold-leave-active{
			  transform: translate3d(0,0,0);
			}
			.list-header{
				height: 40px;
				line-height: 40px;
				padding:0 18px;
				background:#f3f5f7;
				border-bottom: 1px solid rgba(7,17,27,0.1);
				.title{
					float:left;
					font-size: 14px;
					color:rgb(7,17,27);
				}
				.empty{
					float:right;
					font-size: 12px;
					color:rgb(0,160,220);
				}
			}
			.list-content{
				padding:0 18px;
				max-height: 217px;
				overflow: hidden;
				background: #fff;
				.food{
					position: relative;
					padding:12px 0;
					box-sizing: border-box;
					.borderpx(rgba(7,17,27,0.1));
					.name{
						line-height: 24px;
						font-size: 14px;
						color:rgb(7,17,27);
					}
					.price{
						position: absolute;
						right:90px;
						bottom:12px;
						line-height: 24px;
						font-weight: 700;
						font-size: 14px;
						color:rgb(240,20,20);
					}
					.cartcontrol-wrapper{
						position: absolute;
						right:0;
						bottom: 6px;
					}
				}
			}
			
		}
	}
</style>