<template>
    <div class="taskDetail">
        <div class="current">
            <el-card class="box-card">
                <!--头部-->
                <div slot="header" class="clearfix">
                    <h2 style="line-height: 36px;color: #444;width:80%;margin:0 auto;overflow: hidden;text-overflow: ellipsis;white-space: nowrap;">{{'任务名称：' + item.title}}</h2>
                </div>
                <!--任务详情-->
                <div class="text item">
                    <div>发布日期：<span>{{ item.created_at }}</span></div>
                    <div>任务类型：<span>{{ item.work_type }}</span></div>
                    <div> 对口科室：<span>{{ item.department }}</span></div>
                    <div>截止日期：<span>{{ item.end_time }}</span></div>
                    <div>责任人：<span>{{leading}}</span></div>
                    <p class="content"><span style="color: #aaa;">任务内容：</span><span style="max-width=100%;">{{ item.detail }}</span></p>
                </div>
                <!--操作按钮-->
                <div class="seal">
                    <!-- <span v-if="item.finished_at">已完成</span> -->
                   <el-card class="assessBox el-col-16 el-col-push-4">
                             <div v-if="taskPro.assess">
                             <h4 style="margin-bottom:10px">评分结果</h4>
                        <p class="scoreRes">{{'完成情况：&emsp;' + taskPro.quality}}</p>
                        <p class="scoreRes">考核等级：&emsp;<el-tag style="font-size: 15px;" :type="color">{{taskPro.assess.title}}</el-tag></p>

                         <p v-if="taskPro.remark !== null" class="scoreRes">{{'备&emsp;&emsp;注：&emsp;' + taskPro.remark}}</p>
                    </div>
                    <div style="padding-bottom: 20px;" class="el-col-8 el-col-push-8" v-else>
                        <span>尚未评分</span>
                    </div>
                        </el-card>
                </div>
            </el-card>
        </div>
    </div>
</template>
<script>
    export default{
        data () {
            return {
                leading: [],
                color: '',
                //任务详情
                item: [],
                //获取各学院可选责任人
                users: [],
                //是否显示dialog
                isDia: false,
                isAllot: false,
                //是否显示评分结果
                isScores: false,
                //催交情况
                remind: 0,
                //当前选中一级菜单
                currOption: [],
                //当前选中责任人ID
                allot: null,
                taskPro: [],
                //任务提交时是否过了截止日期
                delay: {
                    delayReson: '',
                    isDelay: false,
                    delayMessage: ''
                },
                formLabelWidth: '210px',
                formLabelWidth2: '100px',
                //责任人级联选择器一级菜单
                options: [
                    {
                        name: '全体人员', id: 'all'
                    },
                    {
                        name: '具体单一', id: 'only',
                        children: [
                        ]
                    }
                ],
                prop: {
                    label: 'name',
                    value: 'id'
                }
            }
        },
        methods: {
            //随机生成颜色
            isColor () {
                var color = [
                    'success','warning', 'primary', 'danger', 'success','warning', 'primary', 'danger', 'warning', 'success']
                this.color = color[Math.floor(Math.random()*10)]
            },
            //获取当前选项
            current(){
                if(this.currOption[1]){
                    this.allot = this.currOption[1]
                } else {
                    this.allot = this.currOption[0]
                }
            },
            //判断提交时间是否过了截止日期
            goSubmit () {
                this.$confirm('提交任务后将无法取消, 是否继续?', '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning'
                }).then(() => {
                    this.$http.get('is_delay/' + this.$route.params.id).then(res => {
                        if (res.data.isDelay) {
                            this.delay.delayMessage = '此任务已经过了截止日期，请填写推迟理由后提交！'
                            this.delay.isDelay = true
                        } else {
                            this.isSubmit()
                        }
                    })
                }).catch(err => {
                            for(let i in err.response.data.errors){
                                this.$message({
                                  type: 'error',
                                  message: err.response.data.errors[i]
                              })  
                            }
                                                         
                          })
            },
            //去提交任务
            isSubmit(val){
                this.$http.post('submit_task/' + this.$route.params.id,{
                    delay: val
                }).then(res=>{
                    this.$message.success('提交成功,此任务已完成!')
                    this.$router.push({name: 'tasks_of_college'})
                }).catch(res => {
                    this.$message.error(res.message)
                })
            },
            //指定责任人
            appoint () {
                this.$http.post('create_allot_task', {
                    task_id: this.$route.params.id,
                    user_id: this.allot
                }).then(res => {
                    this.isAllot = true
                    this.isDia = false
                    this.loadItem()
                    this.$message.success('指定成功')
                }).catch(err => {
                            for(let i in err.response.data.errors){
                                this.$message({
                                  type: 'error',
                                  message: err.response.data.errors[i]
                              })  
                            }
                                                         
                          })
            },
            //获取任务详情()
            loadItem () {
            this.$http.get('task/' + this.$route.params.id + '?include=task_progresses&college='+this.$route.params.college).then(res => {
                this.item = res.data.data;
                this.taskPro = this.item.task_progresses.data[0];
                    let tempArr = new Array();
                    if(this.taskPro.leading_official){
                        for(let j in this.taskPro.leading_official){
                            tempArr.push(this.taskPro.leading_official[j].nickname);
                        }
                        this.leading = tempArr.join('、');
                    } else {
                        this.leading = String(this.leading);
                        this.leading = '尚未指定';
                    }
            })
            },
            //获取学院所有用户
            getUsers () {
                this.$http.get('users').then(res => {
                    this.options[1].children = res.data.users
                })
            }
        },
        beforeRouteUpdate (to, from, next) {
            next();
            this.loadItem()
            this.getUsers()
        },
        mounted () {
            this.loadItem()
            this.getUsers()
            this.isColor()
        }
    }
</script>
<style scoped>
    .box-card{
        margin-top:20px;
        min-height:450px;
        padding-bottom:30px;
        position:relative;
    }
    .current>p{
        margin-top:10px;
        text-align:left;
    }
    .scoreBox{
        padding:20px;
        border-radius:5px;
        border:1px solid #eee;
        box-shadow:inset 0 0 5px 5px #eee;
        background:#f5f5f5;
    }
    .taskDetail{
        height:100%;
        width: 100%;
    }
    .clearfix:before,
    .clearfix:after {
        display: table;
        content: "";
    }
    .clearfix:after {
        clear: both
    }
    .assessBox{
        padding:5px 10px;
    }
    .text{
        min-height:200px;
        padding-bottom:50px;
        font-size: 14px;
    }
    .el-form-item>p{
        text-align:left;
    }
    .operaBtn i{
        cursor:pointer;
    }
    .text>div{
        display:inline;
        color:#888;
        margin-right:25px;
    }
    .text>p.content{
        color:#888;
        padding: 15px 30px;
        line-height: 2;
        font-size: 16px;
        text-indent: 2em;
    }
    .text>div span,.text>p span{
        color:#333;
    }
    .el-card__header{
        padding:5px;
    }
    .content{
        text-align:left;text-indent: 2em;word-wrap:break-word;
    }
    .delayMessage{
        color:#FF4949;
        text-align:center;
        font-size:12px;
        padding:10px 0;
    }
    .appoint{
        /*margin-top:50px;*/
    }
    .appo{
        margin-right:100px;
    }
    .el-cascader,.el-input{
        margin-left:-200px;
    }
    .seal>.el-button{
        /*position:absolute;*/
        /*bottom:80px;*/
        /*left:50%;*/
        /*transform: translateX(-50%)*/
    }
    .scoreRes{
        text-align:left;
        color:#444;
        margin-left:20px;
        margin-bottom:10px;
        word-break: break-all;
    }
    .seal>span{
        position:absolute;
        margin:auto;
        top:0;
        bottom:0;
        left:0;
        right:0;
        display:block;
        width:150px;
        height:60px;
        line-height:60px;
        border:10px solid #FF4949;
        font-size:45px;
        color:#FF4949;
        font-weight:bold;
        transform: rotate(25deg);
        animation: play 0.5s ease;
    }
    @keyframes play{
        from{
            transform:scale(5,5);
        }
        to {
            transform:none;
            transform: rotate(25deg);
        }
    }
</style>
