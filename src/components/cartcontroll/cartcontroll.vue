<template>
	<div class="carcontroll">
		<transition name="move">
			<div class="cart-decrease" v-show="food.count>0" @click="decreaseCart">
				<span class="inner icon-remove_circle_outline"></span>
			</div>
		</transition>
		<div class="cart-count" v-show="food.count>0">
			{{food.count}}
		</div>
		<div class="cart-add icon-add_circle" @click="addCart($event)">
			
		</div>
	</div>
</template>
<script type="text/ecmascript-6">
	import Vue from 'vue';
	export default {
		props:{
			food:{
				type:Object
			}
		},
		methods:{
			addCart:function(event){
				if(!event._constructed){
					return;
				}
				if(!this.food.count){
					Vue.set(this.food,'count',1);					
					// this.food.count=1;

				}else{
					this.food.count++;
				}
				this.$root.eventHub.$emit('cart.add', event.target);
			},
			decreaseCart:function(event){
				if(!event._constructed){
					return;
				}
				if(this.food.count){
					this.food.count--;					
					// this.food.count=1;
				}
			}
		}
	}
</script>
<style lang="less">
	.carcontroll{
		font-size: 0;
		.cart-decrease{
			display: inline-block;
			padding:6px;
			transition: all 0.4s linear;
			.inner{	
				line-height: 24px;
				font-size: 24px;
				color: rgb(0,160,220);
				transition: all .5s linear;	
			}
			&.move-enter-active, &.move-leave-active{
				// transition: all .5s linear;
				// transition: all 1s linear;
				opacity: 1;
				transform: translate3d(0,0,0);
				.inner{
					transform: rotate(0);
					display: inline-block;	
				}
			}
			&.move-enter,&.move-leave-active{
				opacity: 0;
				transform:translate3d(24px,0,0);
				.inner{
					transform: rotate(180deg);
				}
			}
		}
		.cart-add{
			display: inline-block;
			vertical-align: top;
			line-height: 24px;
			font-size: 24px;
			color: rgb(0,160,220);
			padding: 6px;
		}
		.cart-count{
			display: inline-block;
			vertical-align: top;
			width: 12px;
			padding-top: 6px;
			line-height: 24px;
			text-align: center;
			font-size: 10px;
			color: rgb(147,153,139);

		}
		.cart-add{
			display: inline-block;
		}
	}
</style>
