<template>
	<el-row style="height: 90vh" >
		<el-col :span="17" align="center" class="left-page">
			<el-scrollbar height="90vh">
				<div ref="info01" style="width: 90%; height: 600px" class="card-like"></div>
				<div ref="info02" style="width: 90%; height: 600px" class="card-like"></div>
			</el-scrollbar>
		</el-col>
		<el-col :span="1"></el-col>
		<el-col :span="6" class="right-page">
			<el-form :model="form" label-width="140px">
				<el-divider content-position="left">基本信息</el-divider>
				<el-form-item label="当前年龄(岁)">
					<el-input-number v-model="form.ageNow" :min="10" :max="200" />
					<el-tooltip
						effect="dark"
						content="计算将从当前年龄开始，当前年龄也算一年时间。"
						placement="top-start"
					>
						<el-icon><InfoFilled /></el-icon>
					</el-tooltip>
				</el-form-item>
				<el-form-item label="退休年龄(岁)">
					<el-input-number v-model="form.ageOver" :min="10" :max="200" />
					<el-tooltip
						effect="dark"
						content="退休年龄当年不参与计算。"
						placement="top-start"
					>
						<el-icon><InfoFilled /></el-icon>
					</el-tooltip>
				</el-form-item>
				<el-form-item label="养老金余额(元)">
					<el-input-number v-model="form.balance" :min="0" :max="1000000" :step="10000" :precision="2"  :controls="false"/>
				</el-form-item>
				<el-form-item label="缴纳基数(元)">
					<el-input-number v-model="form.baseMoney" :min="0" :max="1000000" :step="1000"/>
					<el-tooltip
						effect="dark"
						content="指每月缴纳养老金的基数。当年缴纳基数=此数值*12"
						placement="top-start"
					>
						<el-icon><InfoFilled /></el-icon>
					</el-tooltip>
				</el-form-item>
				<el-form-item label="缴纳基数增长率(%)">
					<el-input-number v-model="form.baseMoneyRate" :min="-100" :max="100" :precision="1" :step="0.5"/>
					<el-tooltip
						effect="dark"
						content="指每年的增长率。下一年缴纳基数=今年缴纳基数*(1+此数值)"
						placement="top-start"
					>
						<el-icon><InfoFilled /></el-icon>
					</el-tooltip>
				</el-form-item>
				<el-form-item label="个人缴纳比例(%)">
					<el-input-number v-model="form.personRate" :min="0" :max="20" :precision="1" :step="0.5"/>
				</el-form-item>
				<el-form-item label="公司缴纳比例(%)">
					<el-input-number v-model="form.companyRate" :min="0" :max="100" :precision="1" :step="0.5"/>
				</el-form-item>
				<el-divider content-position="left">额外信息</el-divider>
				<el-form-item label="养老金利率(%)">
					<el-input-number v-model="form.pensionRate" :min="0" :max="100" :precision="2" :step="0.1"/>
					<el-tooltip
						effect="dark"
						content="指养老金每年的利率。当年利息=养老金余额*此数值。如不清楚，大抵按默认值4.25%即可。"
						placement="top-start"
					>
						<el-icon><InfoFilled /></el-icon>
					</el-tooltip>
				</el-form-item>
				<el-form-item label="通货膨胀率(%)">
					<el-input-number v-model="form.inflationRate" :min="-100" :max="100" :precision="2" :step="0.1"/>
					<el-tooltip
						effect="dark"
						content="通缩请填负数。参考网站https://zh.tradingeconomics.com/china/inflation-cpi"
						placement="top-start"
					>
						<el-icon><InfoFilled /></el-icon>
					</el-tooltip>
				</el-form-item>



				<el-form-item>
					<el-button type="primary" @click="onSubmit">计算</el-button>
					<el-button @click="onReset">重置</el-button>
				</el-form-item>
			</el-form>
		</el-col>
	</el-row>
</template>

<script setup>
import {getCurrentInstance, onMounted, reactive, ref} from "vue";
const { proxy } = getCurrentInstance()
const form = reactive({
	balance:0,
	ageNow:22,
	ageOver:50,
	baseMoney:3000,
	baseMoneyRate:3,
	personRate:8,
	companyRate:20,
	pensionRate:4.25,
	inflationRate:2.05,
})

//图表数据
const dataReset = reactive({
	"info01":{
		"title":"养老金概况-总量",
		"xAxis":[],
		"data":[[],[],[],[],[]],
	},
	"info02":{
		"title":"养老金概况-年量",
		"xAxis":[],
		"data":[[],[],[],[],[]],
	}
})
const data = reactive({
	"info01":{
		"title":"养老金概况-总量",
		"xAxis":[1],
		"data":[[10],[3],[6],[9],[1]],
	},
	"info02":{
		"title":"养老金概况-年量",
		"xAxis":[1],
		"data":[[10],[3],[6],[9],[1]],
	}
})

const reSetData = () =>{
	data.info01 = dataReset.info01;
	data.info02 = dataReset.info02;
	console.log(data)
}


//计算
const onSubmit = () => {
	reSetData();
	let pensionMoney = form.balance;
	for (let i=form.ageNow; i<form.ageOver; ++i){
		//每月基数
		let baseMoneyNew = form.baseMoney * Math.pow(1+form.baseMoneyRate/100, i-form.ageNow);
		//每月个人缴纳量
		let moneyPerMonthPerson = baseMoneyNew * (form.personRate / 100);
		//每月公司缴纳量
		let moneyPerMonthCompany = baseMoneyNew * (form.companyRate / 100);
		//每月总缴纳量
		let moneyPerMonth = moneyPerMonthPerson + moneyPerMonthCompany;
		//今年结束总金额和今年利息
		let [pensionMoneyNew, interestThisYear] = oneYearCalculate(pensionMoney, form.pensionRate/100, moneyPerMonth);
		console.log(pensionMoneyNew, interestThisYear);
		//存入图表数据1
		data.info01.xAxis.push(i);
		data.info01.data[0].push(parseFloat(pensionMoneyNew.toFixed(2)))
		data.info01.data[1].push(parseFloat((moneyPerMonthPerson  * 12 + getLastElementOrDefault(data.info01.data[1])).toFixed(2)))
		data.info01.data[2].push(parseFloat((moneyPerMonthCompany * 12 + getLastElementOrDefault(data.info01.data[2])).toFixed(2)))
		data.info01.data[3].push(parseFloat((getLastElementOrDefault(data.info01.data[1]) + getLastElementOrDefault(data.info01.data[2])).toFixed(2)))
		data.info01.data[4].push(parseFloat((interestThisYear          + getLastElementOrDefault(data.info01.data[4])).toFixed(2)))

		//存入图表数据2
		data.info02.xAxis.push(i);
		data.info02.data[0].push(parseFloat((pensionMoneyNew - pensionMoney).toFixed(2)))
		data.info02.data[1].push(parseFloat((moneyPerMonthPerson  * 12).toFixed(2)))
		data.info02.data[2].push(parseFloat((moneyPerMonthCompany * 12).toFixed(2)))
		data.info02.data[3].push(parseFloat((getLastElementOrDefault(data.info02.data[1]) + getLastElementOrDefault(data.info02.data[2])).toFixed(2)))
		data.info02.data[4].push(parseFloat(interestThisYear.toFixed(2)))

		//update
		pensionMoney = pensionMoneyNew;
	}
	setAllCharts();
}

/*
每个月结束，先计算余额获得的利息，加上每月缴纳数量，累计成新余额。
* */
const oneYearCalculate = (startMoney, rate, moneyPerMonth) =>{
	// console.log(startMoney, rate, moneyPerMonth)
	let monthRate = rate / 12;
	let interestPerYear = 0;
	for (let i=1; i<=12; ++i){
		let interestPerMonth = startMoney * monthRate;
		interestPerYear += interestPerMonth;
		startMoney += interestPerMonth + moneyPerMonth;
		// console.log(interestPerMonth, startMoney)
	}
	return [startMoney, interestPerYear];
}

function getLastElementOrDefault(arr) {
	// 检查数组是否为空
	if (arr.length === 0) {
		return 0;
	} else {
		// 返回最后一个元素
		return arr[arr.length - 1];
	}
}

//重置
const onReset = () => {
	console.log("00")
}


//-----------------------------------------------------------------------------------------------------//
//-----------------------------------------------------------------------------------------------------//
//-----------------                             图表挂载区                            -------------------//
//-----------------------------------------------------------------------------------------------------//
//-----------------------------------------------------------------------------------------------------//
const setAllCharts = () =>{
	setChart01(data.info01)
	setChart02(data.info02)
}
//-----------------                         图表1 - 养老金余额总                        -------------------//
const info01 = ref();
const setChart01 = (dataOne)=>{
	var infoEl = info01.value;//获取ref="info"的标签组件
	var userEc = proxy.$echarts.init(infoEl, "light");//proxy.$echarts是在获取全局配置中的echarts，这样就不需要在每个vue中import echarts了

//此处为图表配置区，可参考Echarts官网替换各种图表
	var option = {
		title: {
			text: dataOne.title
		},
		tooltip: {
			trigger: 'axis',
			axisPointer: {
				type: 'cross',
				label: {
					backgroundColor: '#6a7985'
				}
			}
		},
		legend: {
			data: ['余额总量', '个人缴纳总量', '公司缴纳总量', '缴纳总量', '利息总量']
		},
		toolbox: {
			feature: {
				saveAsImage: {}
			}
		},
		grid: {
			left: '3%',
			right: '4%',
			bottom: '3%',
			containLabel: true
		},
		xAxis: [
			{
				type: 'category',
				boundaryGap: false,
				data: dataOne.xAxis
			}
		],
		yAxis: [
			{
				type: 'value'
			}
		],
		series: [
			{
				name: '余额总量',
				type: 'line',
				label: {
					show: true,
					position: 'top'
				},
				data: dataOne.data[0]
			},
			{
				name: '个人缴纳总量',
				type: 'line',
				data: dataOne.data[1]
			},
			{
				name: '公司缴纳总量',
				type: 'line',
				data: dataOne.data[2]
			},
			{
				name: '缴纳总量',
				type: 'line',
				data: dataOne.data[3]
			},
			{
				name: '利息总量',
				type: 'line',
				data: dataOne.data[4]
			}
		]
	};
	userEc.setOption(option);//将图标挂载到标签组件
}

const info02 = ref();
const setChart02 = (dataOne)=>{
	var infoEl = info02.value;//获取ref="info"的标签组件
	var userEc = proxy.$echarts.init(infoEl, "light");//proxy.$echarts是在获取全局配置中的echarts，这样就不需要在每个vue中import echarts了

//此处为图表配置区，可参考Echarts官网替换各种图表
	var option = {
		title: {
			text: dataOne.title
		},
		tooltip: {
			trigger: 'axis',
			axisPointer: {
				type: 'cross',
				label: {
					backgroundColor: '#6a7985'
				}
			}
		},
		legend: {
			data: ['今年量', '今年个人缴纳量', '今年公司缴纳量', '今年缴纳量', '今年利息量']
		},
		toolbox: {
			feature: {
				saveAsImage: {}
			}
		},
		grid: {
			left: '3%',
			right: '4%',
			bottom: '3%',
			containLabel: true
		},
		xAxis: [
			{
				type: 'category',
				boundaryGap: false,
				data: dataOne.xAxis
			}
		],
		yAxis: [
			{
				type: 'value'
			}
		],
		series: [
			{
				name: '今年量',
				type: 'line',
				label: {
					show: true,
					position: 'top'
				},
				data: dataOne.data[0]
			},
			{
				name: '今年个人缴纳量',
				type: 'line',
				data: dataOne.data[1]
			},
			{
				name: '今年公司缴纳量',
				type: 'line',
				data: dataOne.data[2]
			},
			{
				name: '今年缴纳量',
				type: 'line',
				data: dataOne.data[3]
			},
			{
				name: '今年利息量',
				type: 'line',
				data: dataOne.data[4]
			}
		]
	};
	userEc.setOption(option);//将图标挂载到标签组件
}

onMounted(() =>{
	setAllCharts()
})

</script>


<style scoped>

:deep(.left-page){
	padding: 20px;
	border-radius: 10px;
	background-color: #00000000;
	box-shadow: 3px 0px 5px 3px #00000005, -3px 0px 5px 3px #00000005, 0px 3px 5px 3px #00000005, 0px -3px 5px 3px #00000005;
}

:deep(.right-page){
	padding: 20px;
	border-radius: 10px;
	background-color: #00000000;
	box-shadow: 3px 0px 5px 3px #00000005, -3px 0px 5px 3px #00000005, 0px 3px 5px 3px #00000005, 0px -3px 5px 3px #00000005;
}

:deep(.card-like){
	margin-top:2vh;
	padding: 20px;
	border-radius: 10px;
	box-shadow: 3px 0px 5px 3px #00000005, -3px 0px 5px 3px #00000005, 0px 3px 5px 3px #00000005, 0px -3px 5px 3px #00000005;
}

</style>