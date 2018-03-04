<template>
	<div class="goods">
		<div class="menu-wrapper" ref="menuWrapper">
			<ul>
				<li v-for="(item,index) in goods" class="menu-item" :class="{'current':currentIndex===index}" @click="selectMenu(index,$event)">
					<span class="text">
						<span v-show="item.type>0" class="icon" :class="classMap[item.type]"></span>
						{{item.name}}
					</span>
				</li>
			</ul>
		</div>
		<div class="foods-wrapper" ref="foodsWrapper">
			<ul>
				<li v-for="item in goods" class="food-list food-list-hook">
					<h1 class="title">{{item.name}}</h1>
					<ul v-for="food in item.foods" class="food-item">
						<div class="icon">
							<img :src="food.icon">
						</div>
						<div class="container">
							<h2 class="name">{{food.name}}</h2>
							<p class="desc">{{food.description}}</p>
							<div class="extra">
								<span class="count">月售{{food.sellCount}}份</span><span>好评率{{food.rating}}%</span>
							</div>
							<div class="price">
								<span class="now">¥{{food.price}}</span>
								<span v-show="food.oldPrice" class="old">¥{{food.oldPrice}}</span>
							</div>
							<div class="cartcontroll-wrapper">
								<cartcontroll :food="food"></cartcontroll>
							</div>
						</div>
					</ul>
				</li>
			</ul>
		</div>
		<ShopCart  :delivery-price="seller.deliveryPrice" :min-price="seller.minPrice" :selectFoods="selectFoods"></ShopCart>
	</div>
	
</template>
<script type="text/ecmascript-6">
	const ERR_OK=0;
	import BScroll from 'better-scroll'
	import ShopCart from '../shopcart/shopcart.vue'
	import cartcontroll from '../cartcontroll/cartcontroll.vue'

	export default{
		props:{
			seller:{
				type:Object
			}
		},
		data(){
			return {
				goods:[],
				listHeight:[],
				scrollY:0,
				selectedFood: ''
			};
		},
		computed:{
			currentIndex(){
				for(let i=0;i<this.listHeight.length;i++){
					let height1=this.listHeight[i];
					let height2=this.listHeight[i+1];
					if(!height2 || (this.scrollY>=height1 && this.scrollY<height2)){
						return i;
					}		
				}
				return 0;
			},
			selectFoods(){
				let foods=[];
				this.goods.forEach((good)=>{
					good.foods.forEach((food)=>{
						if(food.count){
							foods.push(food);
						}
					})
				})
				return foods;
			}
		},
		created(){
			this.classMap=['decrease','discount','special','invoice','guarantee'];
			this.$http.get('/api/goods').then((response)=>{
				response=response.body;
				if(response.errno===ERR_OK){
					this.goods=response.data;
					this.$nextTick(()=>{
						this._initScroll();
						this._calculateHeight();
					});		
				}
			})
		},
		methods:{
			_initScroll(){
				this.menuScroll=new BScroll(this.$refs.menuWrapper,{
					click:true
				});
				this.foodsScroll=new BScroll(this.$refs.foodsWrapper,{
					click:true,
					probeType:3
				});
				this.foodsScroll.on('scroll',(pos)=>{
					this.scrollY = Math.abs(Math.round(pos.y));

				});
			},
			_calculateHeight(){
				let footList=this.$refs.foodsWrapper.getElementsByClassName('food-list-hook');
				let height=0;
				this.listHeight.push(height);
				for(let i=0;i<footList.length;i++){
					let item=footList[i];
					height+=item.clientHeight;
					this.listHeight.push(height);
				}
			},
			selectMenu(index,event){
				if(!event._constructed){
					return;
				}
				let foodList=this.$refs.foodsWrapper.getElementsByClassName('food-list-hook');
				// console.log(index);
				let el=foodList[index];
				this.foodsScroll.scrollToElement(el,300);
			}
			
		},
		components:{
			ShopCart,
			cartcontroll
		}

	}
</script>
<style lang="less">
	@import '../../common/less/mixin.less';
	.icon-common(@num){
		&.decrease{
			.bg-img('goods/decrease_@{num}');
		}
		&.discount{
			.bg-img('goods/discount_@{num}');	
		}
		&.guarantee{
			.bg-img('goods/guarantee_@{num}');
		}
		&.special{
			.bg-img('goods/special_@{num}');
		}
		&.invoice{
			.bg-img('goods/invoice_@{num}')
		}
	}
	.goods{
		display: flex;
		position: absolute;
		top:174px;
		bottom:46px;
		width: 100%;
		overflow: hidden;
		.menu-wrapper{
			flex:0 0 80px;
			width: 80px;
			background-color: #f3f5f7;
			.menu-item{
				display: table;
				height: 54px;
				width: 56px;
				line-height: 14px;
				padding: 0 12px;
				.icon{
					display: inline-block;
					width: 12px;
					height: 12px;
					margin-right: 2px;
					background-size: 12px 12px;
					background-repeat: no-repeat;
					vertical-align: top;
					.icon-common(3);
				}
				.text{
					display: table-cell;
					width: 56px;
					height: 54px;
					vertical-align: middle;
					.borderpx(rgba(7,17,27,0.1));
					font-size: 12px;
				}
				&.current{
					position: relative;
					margin-top: -1px;
					z-index: 10;
					background-color: #fff;
					font-weight: 700;
					.text{
						border: none;
					}
				}
			}
		}
		.foods-wrapper{
			flex: 1;
			.title{
				padding-left: 14px;
				height: 26px;
				line-height: 26px;
				border-left: 2px solid #d9dde1;
				font-size: 12px;
				color:rgb(147,153,159);
				background-color: #f3f5f7;
			}
			.food-item{
				display: flex;
				padding:18px;
				.borderpx(rgba(7,17,27,0.1));
				&:last-child{
					.bordernone();
					margin-bottom: 0;
				}
				.icon{
					flex: 0 0 57px;
					margin-right: 10px;
				}
				.container{
					flex: 1;
					.name{
						font-size: 14px;
						color: rgb(7,17,27); 
						height: 14px;
						line-height: 14px;
						margin: 2px 0 8px 0;
					}
					.desc,.extra{
						line-height: 10px;
						font-size: 10px;
						color: rgb(147,153,159);
					}
					.desc{
						margin-bottom: 8px;
						line-height:12px;
					}
					.extra{
						.count{
							margin-right: 12px;	
						}
					}
					.price{
						font-weight: 700;
						line-height: 24px;
						.now{
							margin-right:8px;
							font-size: 14px;
							color: rgb(240,20,20);
						}
						.old{
							font-size: 10px;
							text-decoration: line-through;
						}
					}
					.cartcontroll-wrapper{
						position: absolute;
						right: 0;
						bottom: 12px;
					}

				}
			}

		}
	}
</style>