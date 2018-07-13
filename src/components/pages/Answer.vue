<template>
  <div>
    <Header
      :title="title"
      :href="href"
      :jumpState="jumpState"
      :alert="alert"
      @child="warning"
    />
    <!--答题-->
    <div class="container answer" v-if="layerToggle&&!message">
      <!--页码-->
      <div class="action flex">
        <div class="tip">
          <span>{{page}}</span>
          <span>/{{totalPage}}</span>
        </div>
        <div class="action-btn flex flex-align-center">
          <a @click="answerCardToggle">
            <img src="../../assets/answer.png" alt="">
            <span>答题卡</span>
          </a>
          <a @click="signToggle" v-for="(item,index) in examlist" v-if="index+1==page">
            <img src="../../assets/sign.png" alt="">
            <span>{{item.sign?'已标记':'标记'}}</span>
          </a>
        </div>
      </div>
      <!--主内容-->
      <div class="content">
        <!--标题-->
        <div class="subject">
          <b>{{examName}}</b>
        </div>
        <!--答案-->
        <div class="result">
          <!--item-->
          <div class="result-item" v-for="(item,index) in examlist" :key="item.ID">
            <div v-if="index+1==page">
              <!--题目-->
              <div class="title">
                {{item.QUESTIONS_CONTENT}}
              </div>
              <!--item-->
              <div class="box-item single" v-if="item.QUESTIONS_TYPE==0">

                <div class="box" v-for="(child,idx) in item.option_content" :key="idx">
                  <input type="radio"
                         :id="'radio'+item.ID+idx"
                         :value="child"
                         v-model="item.ANSWER"
                  />
                  <label :for="'radio'+item.ID+idx">{{child}}</label>
                </div>

              </div><!--单选-->

              <div class="box-item multi" v-if="item.QUESTIONS_TYPE==1">

                <div class="box" v-for="(child,idx) in item.option_content" :key="idx">
                  <input type="checkbox"
                         :id="'checkbox'+item.ID+idx"
                         :value="child"
                         v-model="item.ANSWER"
                  />
                  <label :for="'checkbox'+item.ID+idx">{{child}}</label>
                </div>
              </div><!--多选-->
            </div>
          </div>
        </div>

      </div>
      <!--控制器-->
      <div class="submit flex flex-justify-center">
        <div>
          <button :class="page>1?'':'no'" @click="prev">上一题</button>
        </div>
        <div v-if="page<totalPage">
          <button :class="page<totalPage?'':'no'" @click="next">下一题</button>
        </div>
        <div v-if="page==totalPage">
          <button @click="submit">交卷</button>
        </div>
      </div>
    </div>

    <!--答题卡-->
    <div class="container answerCard" v-if="!layerToggle&&!message">
      <div class="subject flex">
        <b>
          党的十九大精神学习试题
        </b>
        <a @click="answerCardToggle">&times;</a>
      </div>
      <div class="guide">
        <dl class="clr">
          <dt>单选题</dt>

          <dd class="lt" v-for="(item,index) in examlist" v-if="item.QUESTIONS_TYPE==0">
            <span @click="guide(index)" :data-index="index+1"
                  :class="[(item.ANSWER.length||item.ANSWER!='')?'sel':'', item.sign?'sign':'']">{{index+1}}</span>
          </dd>
        </dl>
      </div>
      <div class="guide">
        <dl class="clr">
          <dt>多选题</dt>
          <dd class="lt" v-for="(item,index) in examlist" v-if="item.QUESTIONS_TYPE==1">
            <span @click="guide(index)" :data-index="index+1"
                  :class="[(item.ANSWER.length||item.ANSWER!='')?'sel':'', item.sign?'sign':'']">{{index+1}}</span>
          </dd>
        </dl>
      </div>
      <div class="submit">
        <button @click="submit">交卷并查看结果</button>
      </div>
    </div>

    <!--loading-->
    <Loading v-if="loading"/>

    <!--warning-->
    <!--<div class="modal warning" v-if="message">-->
    <!--<div class="modal-diag">-->
    <!--<div class="modal-bg">-->
    <!--<p>你有一次考试没提交，是否退出考试?</p>-->
    <!--</div>-->
    <!--<div class="btn-group">-->
    <!--<button @click="close">退出</button>-->
    <!--<button @click="back">取消</button>-->
    <!--</div>-->
    <!--</div>-->
    <!--</div>-->

  </div>
  <!-- <div class="modal" v-if="result" @click="resultToggle">
     <div>
       <h4>{{message}}</h4>
     </div>
   </div>-->
</template>

<script>
  import Header from '../common/Header';
  import Loading from '../common/Loading';
  import {request} from '../../api/common'
  import {times} from '../../api/time'
  import browser from '../../api/checkVersion';

  export default {
    name: 'Answer',
    components: {
      Header,
      Loading,
    },
    data() {
      return {
        title: '考试',
        href: -1,
        jumpState: true,
        alert: true,
        layerToggle: true,//true为默认答题状态
        totalPage: 0,//总页数
        page: 1,//当前页码
        user_id: '',
        user_name: '',
        integral: '',
        sessionId: '',
        examlist: [],//题目集合
        examName: '同济党建知识竞赛学习试题',//标题
        exampass: 0,//通过分数线
        loading: true,//加载动画
        result: false,//得分页状态
        message: '',
        endTime: '',
        startTime: '',
        state: false,
        clo: true,
        version: '',
        count:0,
      }
    },
    beforeRouteEnter(to, from, next) {
      console.log(to)
      next(vm => {
        if (!vm.global.isLogin) {
          console.log(vm.$router)
          vm.$router.push({path: 'login'})
        }
        else {
          next();

        }
      });
    },
    beforeRouteLeave(to, from, next) {
      console.log(to.path);
      console.log(from.path);
      console.log(this.$router);
      console.log(history);
      console.log('count:'+this.count)
      var alerts = e => {
        console.log(e);
        if (!e.index) {
          next();
        }
        else {
          this.global.isLogin = true;
          next(false);
        }
      };

      if (to.path == "/main" && this.global.isLogin&&!this.count) {
        console.log(this.global.isLogin);
        // next();
        // this.$router.push({ path: "login" });
        mui.confirm("本次答题没提交是否退出", "", ["是", "否"], alerts, "div");
      } else {
        next();
      }

      // next();
      // 导航离开该组件的对应路由时调用
      // 可以访问组件实例 `this`
    },
    created: function () {
      // console.log(__BASE64)
      console.log(this.$route.query)
      this.user_id = this.$route.query.user_id;
      this.user_name = this.$route.query.user_name;
      this.integral = this.$route.query.integral;
      console.log('user_id:' + this.user_id)
      console.log('user_id:' + this.integral)
      this.getAnswercnt();
    },
    mounted() {
      var self = this;
      // self.pushHistory();
      mui.back = function () {

        /*函数体*/
        alert('是否要退出考试')
      };

    },
    methods: {
      pushHistory() {
        var state = {
          title: "answer",
          url: ""
        };
        //window.history.pushState(state, "answer", "");
      },
      warning() {
        var self = this;
        this.$router.go(-1);
        // mui.confirm('本次答题没提交是否退出', '退出', ['是', '否'], self.alerts, 'div');
      },
      alerts(e) {
        this.count = 1;
        !e.index && this.$router.go(-1);
      },
      getAnswercnt(){
        var self = this;
        request.getServerData(
          {
            user_id:this.user_id
          },
          "onlineExamination.action.examinationAction[getAnswercnt]",
          true,
          (result) => {
            console.log(result)
            console.log(result.responsesnum.RESPONSESNUM)
            if(result.responsesnum.RESPONSESNUM=='1'){//今天次数用完了
              this.count = 1;
              mui.alert('今日答题次数已用完明天再来', '', '返回', self.$router.go(-1), 'div');
            }
            else{
              this.getClassExamlist(this.user_id);
            }
          });
      },
      getClassExamlist(user_id) {
        request.getServerData(
          {user_id: user_id},
          "onlineExamination.action.examinationAction[getExaminationPaper]",
          true,
          (result) => {
            console.log(result)
            this.startTime = times();
            if (result.msg) {/*次数用完*/
              this.loading = false;
              this.message = result.msg;
            }
            else {
              if (result.multipleSelectionList.length) {
                result.multipleSelectionList.map(val => {
                  result.singleSelectionList.push(val)
                })
              }
              result.singleSelectionList.map(val => {
                val.startTime = this.startTime;
                val.QUESTIONS_TYPE == 0 ? (val.ANSWER = '') : (val.ANSWER = []);
                val.sign = false;
              })
              this.examlist = result.singleSelectionList;
              console.log(this.examlist)
              this.totalPage = this.examlist.length;
              this.loading = false;
            }
          });

      },
      common(data) {
        this.examlist = data;
        this.totalPage = data.length;
        // this.examName = data[0].answerList[0].category;

        this.loading = false;
        this.setSession(data);
        console.log(data)
      },
      guide(e) {
        console.log(e)
        this.layerToggle = !this.layerToggle;
        this.page = e + 1;
      },
      submit() {
        this.endTime = times();
        var self = this;
        console.log(this.examlist);
        this.loading = true;
        this.examlist.map(val => {
          console.log(val.QUESTIONS_TYPE)
          val.endTime = this.endTime;
          if (val.QUESTIONS_TYPE == 1) {//多选
            console.log(val.ANSWER.length)
            if (val.ANSWER.length > 1) {
              val.ANSWER = Array.prototype.join.call(val.ANSWER, '|');
            }
            else if(val.ANSWER.length==0){
              val.ANSWER = '';
            }
            else{
              val.ANSWER = Array.prototype.join.call(val.ANSWER, '');
            }
          }
        });
        console.log(this.examlist)
        console.log(this.user_id)
        console.log(this.user_name)
        console.log(this.integral)
        // this.setSession(this.examlist)

        request.getServerData(
          {
            examinationPaperList: this.examlist,
            user_id: this.user_id,
            user_name: this.user_name,
            integral: this.integral,
            start_time: this.startTime,
            end_time: this.endTime
          },
          "onlineExamination.action.examinationAction[markingPapers]",
          true,
          (result) => {
            console.log(result)
            if (result == '请求失败') {
              mui.alert('服务器出错', '', '返回', self.back, 'div')
            }
            else {
              var res = JSON.stringify(result);
              this.$router.push({
                path: 'answerResult',
                query: {res}
              })
            }

            // this.$router.go(-1);
            // var res = result;
            // localStorage.removeItem([this.classid]);
            // res.state = this.state;
            // var resStr = JSON.stringify(res)
            // this.$router.push({
            //   path: 'answerResult',
            //   query: {resStr}
            // });
          })


      },
      setSession(val) {
        localStorage.setItem([this.classid], JSON.stringify(val));
      },
      getSession(classid) {
        return JSON.parse(localStorage.getItem([classid]));
      },
      answerCardToggle() {
        this.layerToggle = !this.layerToggle;
        this.setSession(this.examlist);
      },
      close() {
        this.$router.go(-1);
        // if (this.version == 'ios') {
        //   exitCurrentPage();
        //
        // } else if(this.version == 'android') {
        //
        //   _ActionJS.exitCurrentPage();
        // }

        /*alert('state:' + this.state, 'tip:' + this.tip)
        this.setSession(this.examlist);
        if (this.version == 'android' && (!this.tip || !this.state)) {
          alert('关闭')
          _ActionJS.exitCurrentPage();
        }
        else if (this.version == 'ios' && (!this.tip || !this.state)) {
          exitCurrentPage();
        }
        else {
          alert('列表')
          this.$router.go(-1);
        }*/
      },
      back() {
        // this.setSession(this.examlist);
        // this.message = false;
        this.$router.go(-1);
      },
      /* resultToggle() {
         this.result = !this.result;
         this.$router.push('/')
       },*/
      signToggle() {
        console.log(this.page)
        this.examlist[this.page - 1].sign = !this.examlist[this.page - 1].sign;
        this.setSession(this.examlist);
      },
      next() {
        if (this.page < this.totalPage) {
          this.page++;
          this.setSession(this.examlist);
        }
      },
      prev() {
        if (this.page > 1) {
          this.page--;
          this.setSession(this.examlist);
        }
      },
    }
  }
</script>

<style scoped lang="less">
  @import '../../variable';

  .container {
    width: @max;
    height: @max;
    padding-top: 88px;
    box-sizing: border-box;
  }

  .answer {
    padding-bottom: 152px;
    .action {
      padding: 0 24px;
      box-sizing: border-box;
      border-bottom: 1px solid @answer-col-c9; /*no*/
      .tip {
        flex-grow: 1;
        .base(@answer-col-62, @answer-col-62);
        line-height: 88px;
        span {
          .base(@theme, @answer-font-32);
        }
        span + span {
          .base(@answer-col-62, @answer-font-24);
        }
      }
      .action-btn {
        flex-grow: 1;
        justify-content: flex-end;
        a {
          width: 138px;
          height: 58px;
          border: 1px solid @answer-col-c9; /*no*/
          border-radius: 3px; /*no*/
          padding-left: 22px;
          margin-left: 10px;
          img {
            width: 26px;
            height: 33px;
            vertical-align: middle;
          }
          span {
            padding-left: 10px;
            vertical-align: middle;
            line-height: 58px;
            .base(@answer-col-62, @answer-font-20);
          }
        }
      }
    }

    .content {
      padding: 0 24px;
      box-sizing: border-box;
      .subject {
        b {
          .base(@answer-col-62, @answer-font-32);
          line-height: 88px;
          width: 728px;
          overflow: hidden;
          text-overflow: ellipsis;
          white-space: nowrap;
        }
      }
      .result {
        padding-top: 46px;
        box-sizing: border-box;
        .title {
          .base(@answer-col-62, @answer-font-28);
          line-height: 56px;
        }
        .single {
          .box {
            width: 100%;
            min-height: 69px;
            padding: 15px 0;
          }

          input {
            display: none;
          }

          label {
            width: 100%;
            height: 100%;
            display: inline-block;
            position: relative;
            line-height: 69px;
            .base(@answer-col-62, @answer-font-28);
            padding-left: 100px;
            box-sizing: border-box;
          }
          label:active {
            background: #EEEEEE;
          }
          label:after {
            content: ""; /*必须设置*/
            display: inline-block;
            width: 70px;
            height: 70px;
            border: 1px solid @answer-col-c9; /*no*/
            position: absolute;
            top: 0px;
            left: 15px;
            border-radius: 50%;
            box-sizing: border-box;
          }

          input:checked + label:after {
            /*background-color: @theme;*/
            background: url(../../assets/raido.png) no-repeat;
            background-size: 100%;
          }
        }
        .multi {
          .box {
            width: 100%;
            min-height: 70px;
            padding: 15px 0;
          }
          input {
            display: none;
          }
          label {
            width: 100%;
            height: 100%;
            display: inline-block;
            line-height: 75px;
            position: relative;
            .base(@answer-col-62, @answer-font-28);
            padding-left: 100px;
            box-sizing: border-box;
          }
          label:active {
            background: #EEEEEE;
          }
          label:after {
            content: "";
            width: 71px;
            height: 71px;
            /*border: 1px solid @answer-col-c9; !*no*!*/
            display: inline-block;
            position: absolute;
            top: 0;
            left: 15px;
            background: url(../../assets/no-multi.png) no-repeat;
            background-size: 100%;
          }
          input:checked + label:after {
            content: "";
            /* color: @white;
             text-align: center;
             font-size: 60px;*/
            background: url(../../assets/multi.png) no-repeat;
            background-size: 100%;
          }

        }
      }
    }

    .submit {
      position: fixed;
      bottom: 0;
      background-color: @white;
      width: @max;
      height: 152px;
      padding-top: 32px;
      box-sizing: border-box;
      text-align: center;
      div {
        flex-grow: 1;
        button {
          background-color: @theme;
          border-radius: 5px; /*no*/
          border: 0;
          width: 258px;
          line-height: 88px;
          .base(@white, @answer-font-30);
        }
        button.no {
          color: @answer-col-62;
          background-color: @white;
          border: 1px solid @answer-col-c9; /*no*/
        }
      }

    }
  }

  .answerCard {
    padding-bottom: 152px;
    .subject {
      padding: 0 27px;
      box-sizing: border-box;
      border-bottom: 1px solid @answer-col-c9; /*no*/
      b {
        .base(@answer-col-62, @answer-font-32);
        line-height: 88px;
        width: 728px;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
      }
      a {
        display: inline-block;
        font-size: 50px;
        height: 50px;
        line-height: 88px;
      }
    }
    .guide {
      padding-top: 36px;
      dl {
        width: @max;
        dt {
          .base(@answer-col-62, @answer-font-28);
          padding: 0 24px;
          box-sizing: border-box;

        }
        dd {
          .base(@answer-col-62, @answer-font-28);
          width: 20%;
          height: 108px;
          display: flex;
          justify-content: center;
          align-items: center;
          span {
            display: block;
            border-radius: 50%;
            border: 4px solid @answer-col-c9; /*no*/
            .base(@answer-col-62, @answer-font-28);
            width: 68px;
            height: 68px;
            line-height: 68px;
            text-align: center;
          }
          span.sel {
            background-color: @theme;
            color: @white;
          }
          span.sign {
            border-color: #1F86FF
          }
        }
      }
    }
    .submit {
      position: fixed;
      bottom: 0;
      background-color: @white;
      width: @max;
      height: 152px;
      padding-top: 32px;
      box-sizing: border-box;
      text-align: center;
      button {
        background-color: @theme;
        border-radius: 5px; /*no*/
        border: 0;
        width: 500px;
        line-height: 88px;
        .base(@white, @answer-font-30);
      }

    }
  }

  .modal {
    position: fixed;
    left: 0;
    top: 0;
    right: 0;
    bottom: 0;
    width: @max;
    height: @max;
    background-color: rgba(0, 0, 0, .4);
    z-index: 1000;
    img {
      position: absolute;
      top: 50%;
      left: 50%;
      width: 300px;
      height: 300px;
      margin-left: -150px;
      margin-top: -150px;
    }
    .modal-diag {
      position: absolute;
      top: 50%;
      left: 50%;
      margin-left: -310px;
      margin-top: -309px;
      .modal-bg {
        width: 620px;
        height: 520px;
        background: url(../../assets/modal-bg.png) no-repeat;
        background-size: cover;
        position: relative;
        p {
          width: 520px;
          color: @theme;
          font-size: 30px;
          position: absolute;
          bottom: 46px;
          left: 50%;
          margin-left: -260px;
          text-align: center;
        }
      }
      .btn-group {
        width: @max;
        height: 98px;
        display: table;
        content: '';
        clear: both;
        button {
          color: @white;
          font-size: 30px;
          background-color: @theme;
          height: 98px;
          line-height: 98px;
          border: 0;
          width: @max/2;
          box-sizing: border-box;
          padding: 0;
          margin: 0;
          float: left;
        }
        button:last-child {
          background-color: #918181;
        }
      }
    }
  }

  /*.modal {
    position: fixed;
    left: 0;
    top: 0;
    right: 0;
    bottom: 0;
    width: @max;
    height: @max;
    background-color: rgba(0, 0, 0, .4);
    z-index: 1000;
    img {
      position: absolute;
      top: 50%;
      left: 50%;
      width: 300px;
      height: 300px;
      margin-left: -150px;
      margin-top: -150px;
    }
    .modal-diag {
      position: absolute;
      top: 50%;
      left: 50%;
      margin-left: -310px;
      margin-top: -309px;
      .modal-bg {
        width: 620px;
        height: 520px;
        background: url(../../assets/modal-bg.png) no-repeat;
        background-size: 100%;
        position: relative;
        p {
          width: 520px;
          color: @theme;
          font-size: 30px;
          position: absolute;
          bottom: 46px;
          left: 50%;
          margin-left: -260px;
          text-align: center;
        }
      }
      .btn {
        width: @max;
        height: 98px;
        button {
          color: @white;
          font-size: 30px;
          background-color: @theme;
          height: 98px;
          line-height: 98px;
          border: 0;
          width: @max;
          box-sizing: border-box;
          padding: 0;
          margin: 0;
        }
      }
    }
  }*/
</style>
