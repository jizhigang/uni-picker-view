# uni-picker-view
uni-app嵌入页面的选择器

开发中需要用到嵌入页面中的时间选择器，于是研究了一下`picker-view`，官方教程：
<https://uniapp.dcloud.io/component/picker-view?id=picker-view>

我需要实现的几点为：
- 默认显示当前时间
- 超过当前时间的日期不显示
- 要显示有效日期

### 一、页面布局与数据

template
```
<template>
	<view class="content">
		<cu-custom bgColor="bg-white" isBack="true" style="width: 750upx;">
			<block slot="backText"></block>
			<block slot="content">选择器</block>
		</cu-custom>
		<picker-view class="mpvue-picker-view" v-if="visible" :indicator-style="indicatorStyle" :value="value" @change="bindChange">
		    <picker-view-column>
		        <view class="item" v-for="(item,index) in years" :key="index">{{item}}年</view>
		    </picker-view-column>
		    <picker-view-column>
		        <view class="item" v-for="(item,index) in months" :key="index">{{item}}月</view>
		    </picker-view-column>
		    <picker-view-column>
		        <view class="item" v-for="(item,index) in days" :key="index">{{item}}日</view>
		    </picker-view-column>
		</picker-view>
	</view>
</template>
```


数据源
```
		data() {
		    const date = new Date()
		    const years = []
		    const year = date.getFullYear()
		    const months = []
		    const month = date.getMonth() + 1
		    const days = []
		    const day = date.getDate()
		    for (let i = 2010; i <= date.getFullYear(); i++) {
		        years.push(i)
		    }
		    for (let i = 1; i <= date.getMonth()+1; i++) {
		        months.push(i)
		    }
		    for (let i = 1; i <= date.getDate(); i++) {
		        days.push(i)
		    }
			return {
		        title: 'picker-view',
		        years,
		        year,
		        months,
		        month,
		        days,
		        day,
		        value: [9999, month - 1, day - 1],
		        visible: true,
		        indicatorStyle: `height: ${Math.round(uni.getSystemInfoSync().screenWidth/(750/100))}px;`,
				babyAge:0,
				currentYear: 2019,
				currentMonth: 1,
				currentDay: 1
			}
		},
		onLoad() {
			var date = new Date()
			this.currentYear = date.getFullYear()
			this.currentMonth = date.getMonth()+1
			this.currentDay = date.getDate()
					
			console.log('当前年'+date.getFullYear())
			console.log('当前月'+(date.getMonth()+1).toString())
			console.log('当前日'+date.getDate().toString())
			console.log('当前时'+date.getHours().toString())
			console.log('当前分'+date.getMinutes().toString())
			console.log('当前秒'+date.getSeconds().toString())
		},
```

样式
```
<style lang="less">

	.content{
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		background-color: #FFFFFF;
		height: 100vh;
		width: 100vw;
		
		
		.mpvue-picker-view {
			width: 100%;
			height: 50%;
			background-color: rgba(255, 255, 255, 1);
			
			.item {
			  text-align: center;
			  width: 100%;
			  height: 88upx;
			  line-height: 88upx;
			  text-overflow: ellipsis;
			  white-space: nowrap;
			  font-size: 40upx;
			}
		}
	}
	
</style>

```


### 二、回调方法

```
bindChange: function (e) {
			    const val = e.detail.value
			    this.year = this.years[val[0]]
			    this.month = this.months[val[1]]
			    this.day = this.days[val[2]]
				
				//将选择的年月日变为number形式，便于比较之用
				var y = parseInt(this.year)
				var m = parseInt(this.month)
				var d = parseInt(this.day)
				
				//选择不同月份显示的天数不同
				if(m==1||m==3||m==5||m==7||m==8||m==10||m==12){
					if(this.days.length!=31){
						this.days = []
						for (let i = 1; i <= 31; i++) {
						    this.days.push(i)
						}
					}
				}else if(m==4||m==6||m==9||m==11){
					if(this.days.length!=30){
						this.days = []
						for (let i = 1; i <= 30; i++) {
						    this.days.push(i)
						}
					}
				}else if(m==2){
					if((y%4==0&&y%100!=0)||(y%400==0)){//闰年
						if(this.days.length!=29){
							this.days = []
							for (let i = 1; i <= 29; i++) {
							    this.days.push(i)
							}
						}
					}else{//平年
						if(this.days.length!=28){
							this.days = []
							for (let i = 1; i <= 28; i++) {
							    this.days.push(i)
							}
						}
					}
				}
				
				
				//处理选择今年的情况
				if(y==this.currentYear){
					
					//最多显示到当前月份
					if(this.months.length!=this.currentMonth){
						this.months = []
						for (let i = 1; i <= this.currentMonth; i++) {
						    this.months.push(i)
						}
					}
					
					//如果选择的是当前月份，那么日最多显示到今天
					if(m==this.currentMonth){
						if(this.days.length!=this.currentDay){
							this.days = []
							for (let i = 1; i <= this.currentDay; i++) {
							    this.days.push(i)
							}
						}
					}
					
				}else{
					this.months = []
					for (let i = 1; i <= 12; i++) {
					    this.months.push(i)
					}
				}
				
				
				//我的业务中是选择的孩子日期，根据选择的日期可以计算出孩子几岁了😄
				if(y>=this.currentYear){
					this.babyAge = 0
				}else{
					//选择的月份大于当前月份
					if(m>this.currentMonth){
						this.babyAge = this.currentYear-y-1
					}else if(m==this.currentMonth){
						if(d>=this.currentDay){
							this.babyAge = this.currentYear-y-1
						}else{
							this.babyAge = this.currentYear-y
						}
					}else{
						this.babyAge = this.currentYear-y
					}
				}
				
				
				console.log('当前选中'+this.year.toString()+'年'+this.month.toString()+'月'+this.day.toString()+'号')
			},
```

简书：<https://www.jianshu.com/p/ffef1ba00aa5>
