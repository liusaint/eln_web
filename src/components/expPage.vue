	<template>

		<!-- 第三个页面 选择记录本-->
		<div>

			<div class="top-bar bgwhite">
				<i class="eln-ico left-arrow"  @click="goBack"></i>				
				<div class="txt">选择实验记录</div>
				<!-- 在这里面用$route -->
				<router-link :to="'/searchPageExp/'+$route.params.id" class="eln-ico right-search" tag="i"></router-link>
			</div>	

			<ul class="bgwhite mt6"
			v-infinite-scroll="getList"
			infinite-scroll-disabled="loading"
			infinite-scroll-distance="20"
			>
				<li v-for="(item,index) in expData" @click="chooseExp(item)" class="choose-bar after-ico">
					{{calExpName(item)}}
				</li>
			</ul>
		</div>

	</template>
	<script>
		import { Toast } from 'mint-ui';
		import {ajax,localSave,calDate}  from '../js/common'
		import { mapState } from 'vuex'
		export default {
			data() {
					return {
						expData: [],
						loading: false,
						page: 1,
						book_id: '',
						dataOver: false, //数据是否加载完
					}
				},
				computed: {
					...mapState(['book', 'exp', 'uid']),
				},

				methods: {
					chooseExp(item) {
						this.$store.commit('chooseExp', item);
						//跳转到提交页面。
						// this.$router.replace('/submitPage');
						this.$router.go(-2);
					},
					goBack() {
						//回到记录本页面。
						this.$router.go(-1);
					},
					//计算实验名
					calExpName(exp) {
						var exp_page = exp.exp_page;
						var len = exp_page.length;
						if (1 == len) {
							exp_page = '00' + exp_page;
						} else if (2 == len) {
							exp_page = '0' + exp_page;
						}
						var res = 'N' + (calDate(exp.book_create_time)).getFullYear().toString().slice(2) + exp.code + '-' + exp_page;
						//把计算后的结果保存一下。
						exp.caled_name = res;
						return res;
					},
					getList() {

						var self = this;

						if (this.dataOver) {
							return;
						}
						this.leading = true;
						ajax({
							url: '/eln/exp-list',
							method: 'post',
							data: {
								page: this.page,
								uid: this.uid,
								book_id: this.book_id,
								limit: 30
							},
							callback: function(data) {

								self.loading = false;

								if (1 == data.status) {
									var parsedData = JSON.parse(data.data);
									var expList = parsedData.expList;
									if (1 == self.page) {
										self.expData = expList;
									} else {
										self.expData = self.expData.concat(expList)
									}
									//检查所有数据，来确认。
									if (0 == self.expData.length) {
										Toast('该记录本下没有实验')
									}
									//检查最近一次获取的数据来确认。
									if (expList.length == 0) {
										self.dataOver = true;
									} else {
										self.page++;

										//如果没有出现滚动条。说明数据没加载满。那么需要继续加载。这段代码还要再测试
										// if (!(document.body.style.overflow!="hidden"&&document.body.scroll!="no"&&document.body.scrollHeight>document.body.offsetHeight)){										self.getList();

										// }	
									}

								}

							}
						})
					},
				},
				created() {

					//初始化，拿数据
					this.loading = false;
					this.page = 1;
					this.dataOver = false;
					this.book_id = this.$route.params.id;


				}

		}
	</script>
