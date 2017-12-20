<template>
  <div id="vue">
    <div id="loadingDiv" v-show="loading">
      <div class="loading">Loading ...</div>
    </div>
    <div class="content-header" style="padding:10px">
      <div class="breadcrumb">
        <span>
          <a v-on:click="ApiaClick()">
            <i class="fa fa-home"></i>首页
          </a>
        </span>
        <span>资产管理</span>
        <span class="active">资产入库</span>
      </div>
    </div>
    <div class="content">
      <div class="row">
        <div class="col-md-12 col-sm-12">
          <div class="box">
            <div class="box-header">
              <div class="form-group clearfix">
                <label class="control-label" >搜索</label>
                <div class="search">
                  <input type="text" class="form-control" v-model="search" placeholder="请输入关键词查询" />
                  <i class="fa fa-search" v-on:click="loaddata(1)"></i>
                </div>
              </div>
              <div class="form-group clearfix ">
                <label class="control-label">操作</label>
                <div class="pull-left">
                  <button class="btn btn-primary btn-sm" type="button" v-on:click="addlist()"><i class="fa fa-plus"></i>新建入库单</button>
                  <button class="btn btn-primary btn-sm" type="button"  v-on:click="xiazai"><i class="fa fa-download"></i>模板下载</button>
                  <button class="btn btn-primary btn-sm" type="button" v-on:click="daoru"><i class="fa fa-shopping-basket"></i>批量入库</button>
                </div>
              </div>
            </div>
            <div class="box-body border-top">
              <div class="btn btn-warning margin-bottom" @click="editlist"><i class="fa fa-edit"></i> 编  辑 </div>
              <div style="overflow-x: auto">
                <div id="gridList"></div>
              </div>
              <!--郭超改部分-->
              <!-- <vueTable ref="table"
                        :columns="columns"
                        :url="url"
                        :type="type"
                        :search="search"
                        v-on:selectId="selectId">
              </vueTable> -->
            </div>
          </div>
        </div>
      </div>
    </div>
    <div class="modal fade" id="myModal3" tabindex="-1" role="dialog" aria-labelledby="myModalLabel"
         aria-hidden="true">
      <div class="modal-dialog">
        <div class="modal-content" style="width: 400px; position: absolute; margin-left: -200px; left: 50%;margin-top:160px">
          <div class="modal-header">
            <button type="button" class="close " data-dismiss="modal" aria-hidden="true"
                    v-on:click='updataform'>
              x
            </button>
            <p class="modal-title">批量导入</p>
          </div>
          <div class="modal-body" style="min-height: 60px;">
            <div class="form-group clearfix">
              <input class="pull-left" type="file" accept="" id="filebutt" />
              <button type="button" class="btn btn-primary btn-sm pull-right" v-on:click="update()"><i class="fa fa-upload"></i>上传</button>
            </div>
            <a v-bind:href='this.urldata'></a>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
  var zNodes;
  var componet;
  var istype;
  // import vueTable from "./KdTable";
  import api from "../api/api";

  export default (componet = {
    components: {
      // vueTable
    },
    data() {
      return {
        //郭超改部分
        //传给子组件的列信息
        // 列表选中的值  id
        selectTableData:[],
        usertoken: '',
        columns: [
          {
            selectable: 'row',
            width: "50px"
          },
          {
            title: "资产类别",
            field: "categoryid",
            width: "150px",
            filterable: {
              operators: {
                string: {
                  startswith: "等于和",
                  eq: "包含不等于",
                  neq: "包含等于"
                }
              },
            }
          },
          {
            title: "资产名称",
            field: "assetname",
            width: "150px",
            filterable: {
              'multi': true, //行过滤时是否多选
              'checkAll': true, //行过滤时是否可以全选
            }
          },
          {
            title: "资产编码",
            field: "assetscoding",
            width: "150px"
          },
          {
            title: "入库单号",
            field: "orderid",
            width: "150px"
          },
          {
            title: "二维码ID",
            field: "assetqrcode",
            width: "150px"
          },
          {
            title: "SN号",
            field: "snno",
            width: "150px"
          },
          {
            title: "使用部门",
            field: "department",
            width: "150px"
          },
          {
            title: "使用地",
            field: "useto",
            width: "150px"
          },
          {
            title: "现使用人",
            field: "usename",
            width: "150px"
          },
          {
            title: "入库日期",
            field: "inbounddate",
            width: "150px"
          },
          {
            title: "操作时间",
            field: "createdate",
            width: "150px",
            filterable: {
              ui: "datetimepicker"
            }
          },
        ],
        //传给子组件URL
        url: '',
        //子组件列表选中id
        idList: [],
        //传给子组件的全局搜索值
        search: '',
        //传给子组件的列表类型值
        type: 6,

        assetqrcode: "", //  资产二维码——下拉框
        assetqrcodedata: [], //二维码
        categoryid: "",
        assetname: "", //资产名称
        assetscoding: "", //资产编码
        inbounddate: "", //入库时间
        showbutton: true, //确认删除？
        treeData: [],
        Imageupload: "",
        loading: false,
        urldata: "",
        usename: "", //现使用人
        message: ""
      };
    },
    mounted: function() {
      this.list();
    },
    methods: {

      list: function() {
        var Vue = this;
        var dataSource = new kendo.data.DataSource({
          serverPaging: true,
          serverSorting: true,
          serverFiltering: true,
          pageSize: 10,
          transport: {
            read: {
              /*url: "http://60.205.149.65:8004/api/Wangqiming/Lists",*/
              url: this.url,
              // url: "http://10.116.67.12:65400/api/AcceptanceForm/SolrList",
              type: "post",
              dataType: "json",
              contentType: "application/json; charset=utf-8"
            },
            parameterMap: function(options, operation) {
              if (operation == "read") {
                if (options.skip || options.pageSize) {
                  Vue.page = options.skip;
                  Vue.pageSize = options.pageSize;
                }
                var parameter = {
                  page: Vue.page,
                  pageSize: Vue.pageSize,
                  usertoken: Vue.usertoken,
                  text_search: Vue.search,
                  sort: options.sort,
                  filter: options.filter,
                  asset_type: Vue.type
                };
                return JSON.stringify(parameter);
              }
            }
          },
          schema: {
            model: {
              id: "assetsid"
            },
            //返回值
            data: function(res) {
              return res.data.results;
            },
            //总页数
            total: function(res) {
              return res.data.count;
            }
          }
        });
        $("#gridList").kendoGrid({
          dataSource: dataSource,
          toolbar: "<center>资产入库列表</center>",
          resizable: true, //是否允许用户调整列宽
          batch: true, //是否可以批量处理
          scrollable: true, //是否有滚动条
          persistSelection: true, //是否有缓存
          sortable: true, //是否开启点击列标题排序功能
          reorderable: true, //是否允许用户自己调整列的顺序
          noRecords: true, //列表没有数据时列表中显示一个提示区域
          height: 400, //提示区域自定义大小
          messages: { //提示区域自定义文字
            noRecords: "当前页面上没有数据"
          },
          pageable: {
            input: true, //是否有页码输入框
            numeric: true,
            refresh: true, //是否显示刷新按钮
            pageSize: 2, //每页显示条数
            buttonCount: 3, //显示页码数字，超出显示省略号
            pageSizes: [10, 20, 50, 100], //可选则每页显示多少条
            messages: {
              display: "{0} - {1} 共 {2} 条数据",
              empty: "没有数据",
              page: "页",
              of: "/ {0}",
              itemsPerPage: "条每页",
              first: "第一页",
              previous: "前一页",
              next: "下一页",
              last: "最后一页",
            }
          },
          columnMenu: {
            messages: {
              sortAscending: "升序",
              sortDescending: "降序",
              columns: '列',
              filter: '过滤',
              sort: '排序'
            }
          },
          filterable: {
            'extra': false,
            // 'multi': true, //行过滤时是否多选
            // 'checkAll': true, //行过滤时是否可以全选
            'messages': {
              info: '显示符合一下条件的行',
              cancel: "取消过滤",
              and: "且",
              or: "或",
              filter: "过滤",
              clear: "清除"
            },
            //字符串型
            string: {
              eq: "Is equal to",    //相等
              neq: "Is not equal to",   //不相等
              startswith: "Starts with",  //开头包含
              contains: "Contains", //包含
              doesnotcontain: "Doesn't contain",  //不包含
              endswith: "Ends"    //结尾包含
            },
            //数值型
            number: {
              eq: "Equal to",   //相等
              neq: "Not equal to",    //不相等
              gte: "Greater than or equal to",//大于等于
              gt: "Greater than",//大于
              lte: "Less than or equal to",//小于等于
              lt: "Less than"//小于
            },
            //日期型
            date: {
              gt: "After",
              lt: "Before",
              eq: "Equal",
              neq: "Not equal",
              gte: "After or equal to",
              lte: "Before or equal to"
            },
          },
          change: function() {
            var selectList = this.selectedKeyNames();
            Vue.selectTableData = this.selectedKeyNames();
//            Vue.selectList = selectList;
//            Vue.$emit('selectId', selectList);

          },
          columns: this.columns
        })
      },
      //郭超改部分
      selectId(selectList){
        this.idList = selectList;
      },

      // 加载load事件
      toggleLoading: function (show) {
        this.loading = show;
      },
      xiazai() {
        window.location.href =
          api.domainUrl + "Export/DownloadExcel?time=" + new Date ().getTime ();
        //window.location.href = 'http://"+api.domainUrl+"/upload/入库模版.xls';
      },
      daoru() {
        $ ("#myModal3").modal ("show");
        $ ("#filebutt").val ("");
      },
      updataform() {
        $ ("#filebutt").val ("");
      },
      checkall(id) {
        this.checkeditem = id;
        console.log (this.checkeditem);
      },
      choosemethod(methodname, id) {
        var func = eval ("this." + methodname);
        new func (id);
      },
      // 查询列表
      loaddata() {
        this.list();
      },
      addlist() {
        var url = "LncomingNew.html";
        var width = $ (window).width ();
        var height = $ (window).height ();
        var iWidth = 1200; //弹出窗口的宽度;
        var iHeight = 500; //弹出窗口的高度;
        //获得窗口的垂直位置
        var iTop = (window.screen.availHeight - 30 - iHeight) / 2;
        //获得窗口的水平位置
        var iLeft = (window.screen.availWidth - 10 - iWidth) / 2;
        window.open (
          url,
          "newwindow",
          "height=" +
          iHeight +
          ",width=" +
          iWidth +
          ",top=" +
          iTop +
          ",left=" +
          iLeft +
          ",resizable=yes,scrollbars=1"
        );
      },
      editlist() {
        if(this.selectTableData.length!==1){
          alert('需要选择一项资产')
          return
        }
        let id=this.selectTableData.toString();

        var url = "InventoryChanges.html?assetsid=" + id;
        var width = $ (window).width ();
        var height = $ (window).height ();
        var iWidth = 1000; //弹出窗口的宽度;
        var iHeight = 500; //弹出窗口的高度;
        //获得窗口的垂直位置
        var iTop = (window.screen.availHeight - 30 - iHeight) / 2;
        //获得窗口的水平位置
        var iLeft = (window.screen.availWidth - 10 - iWidth) / 2;
        window.open (
          url,
          "newwindow",
          "height=" +
          iHeight +
          ",width=" +
          iWidth +
          ",top=" +
          iTop +
          ",left=" +
          iLeft +
          ",resizable=yes,scrollbars=1"
        );
      },
      //dellist(){
      dellist2(id) {
        $.MsgBox.Confirm ("提示", "确定要删除此条资产信息？", this.dellist);
        this.message = id;
      },
      dellist() {
        // alert(id);
        var _self = this;
        this.showbutton = false;
        // if (!confirm('确定要删除此模块？')) {
        //     this.showbutton = true;
        //     return false;
        // }
        var httpOptions = {
          timestamp: new Date ().getTime (), //时间戳，防止加载缓存
          //assetsid:'0a4ed123-569a-405f-ab99-519d2720dd1c',
          assetsid: this.message
        };
        api.Assetinfo
          .DelLoad (httpOptions)
          .then (res => {
            if (res.data.flag == 1) {
              if (res.data.success == 1) {
                alert ("删除成功！");
                _self.loaddata (1);
              }
            } else {
              //atoast.error(res.data.error.message);
              alert ("删除失败！");
            }
          })
          .catch (error => {
            console.log (error);
          });
      },
      //资产类别树状方法
      categoryInfoLoad() {
        var _self = this;
        var params = {};
        api
          .test (params)
          .then (res => {
            if (res.data.flag == 1) {
              if (res.data.success == 1) {
                this.treeData = res.data.data;
                zNodes = this.treeData;
                //alert(zNodes);
                _self.$nextTick (function () {
                  $ (document).ready (function () {
                    var tree = $.fn.zTree.init ($ ("#treeDemo"), setting, zNodes);
                    tree.expandAll (true);
                    $ ("#treename").click (function () {
                      //zNodes[0].id
                      console.log (zNodes);
                      if ($ ("#treeDemo").css ("display") == "none") {
                        $ ("#treeDemo").show ();
                      } else {
                        $ ("#treeDemo").hide ();
                      }
                    });
                    $ ("#sel").click (function () {
                      //zNodes[0].id
                      console.log (zNodes);
                      if ($ ("#treeDemo").css ("display") == "none") {
                        $ ("#treeDemo").show ();
                      } else {
                        $ ("#treeDemo").hide ();
                      }
                    });
                  });
                });
              }
            } else {
              atoast.error (res.data.error.message);
            }
          })
          .catch (error => {
            console.log (error);
          });
      },
      beforeClick(treeId, treeNode) {
        var dcsp =
          '<span class="sel" style="float: right;margin-right: 2px;"></span>';
        $ ("#treename").val (treeNode.name);
        $ ("#treeid").val (treeNode.id);
        $ ("#treeDemo").hide ();
        if (treeNode.isParent) {
          return true;
        } else {
          return false;
        }
      },
      AlertBox: function () {
        $.MsgBox = {
          Alert: function (title, msg, msg2) {
            GenerateHtml ("alert", title, msg, msg2);
            btnOk (); //alert只是弹出消息，因此没必要用到回调函数callback
            btnNo ();
          },
          Confirm: function (title, msg, callback) {
            GenerateHtml ("confirm", title, msg);
            btnOk (callback);
            btnNo ();
          }
        };
        //生成Html
        var GenerateHtml = function (type, title, msg, msg2) {
          var _html = "";
          _html +=
            '<div id="mb_box"></div><div id="mb_con"><span id="mb_tit">' +
            title +
            "</span>";
          _html +=
            '<a id="mb_ico">x</a><div id="mb_msg">' +
            msg +
            '</div><div id="mb_btnbox">';
          if (msg2) {
            _html += '<a id="mb_msg2" href=' + msg2 + ">" + "下载错误信息" + "</a>";
          }
          if (type == "alert") {
            _html += '<input id="mb_btn_ok" type="button" value="确定" />';
          }
          if (type == "confirm") {
            _html += '<input id="mb_btn_ok" type="button" value="确定" />';
            _html += '<input id="mb_btn_no" type="button" value="取消" />';
          }
          _html += "</div></div>";
          //必须先将_html添加到body，再设置Css样式
          $ ("body").append (_html);
          GenerateCss ();
        };
        //生成Css
        var GenerateCss = function () {
          $ ("#mb_box").css ({
            width: "100%",
            height: "100%",
            zIndex: "99999",
            position: "fixed",
            filter: "Alpha(opacity=60)",
            backgroundColor: "black",
            top: "0",
            left: "0",
            opacity: "0.6"
          });
          $ ("#mb_con").css ({
            zIndex: "999999",
            width: "400px",
            position: "fixed",
            backgroundColor: "White"
            //borderRadius: '15px'
          });
          $ ("#mb_tit").css ({
            display: "block",
            fontSize: "14px",
            color: "#444",
            padding: "10px 15px",
            backgroundColor: "#DDD",
            //borderRadius: '15px 15px 0 0',
            borderBottom: "3px solid #009BFE",
            fontWeight: "bold"
          });
          $ ("#mb_msg").css ({
            padding: "20px",
            lineHeight: "20px",
            borderBottom: "1px dashed #DDD",
            fontSize: "13px"
          });
          $ ("#mb_msg2").css ({
            display: "block",
            paddingBottom: "10px",
            paddingLeft: "20px",
            textAlign: "left",
            lineHeight: "20px",
            borderBottom: "1px dashed #DDD",
            fontSize: "13px"
          });
          $ ("#mb_ico").css ({
            display: "block",
            position: "absolute",
            right: "10px",
            top: "9px",
            border: "1px solid Gray",
            width: "18px",
            height: "18px",
            textAlign: "center",
            lineHeight: "16px",
            cursor: "pointer",
            borderRadius: "12px",
            fontFamily: "微软雅黑"
          });
          $ ("#mb_btnbox").css ({
            margin: "15px 0 10px 0",
            textAlign: "center"
          });
          $ ("#mb_btn_ok,#mb_btn_no").css ({
            width: "85px",
            height: "30px",
            color: "white",
            border: "none"
          });
          $ ("#mb_btn_ok").css ({
            backgroundColor: "#168bbb"
          });
          $ ("#mb_btn_no").css ({
            backgroundColor: "gray",
            marginLeft: "20px"
          });
          //右上角关闭按钮hover样式
          $ ("#mb_ico").hover (
            function () {
              $ (this).css ({
                backgroundColor: "Red",
                color: "White"
              });
            },
            function () {
              $ (this).css ({
                backgroundColor: "#DDD",
                color: "black"
              });
            }
          );
          var _widht = document.documentElement.clientWidth; //屏幕宽
          var _height = document.documentElement.clientHeight; //屏幕高
          var boxWidth = $ ("#mb_con").width ();
          var boxHeight = $ ("#mb_con").height ();
          //让提示框居中
          $ ("#mb_con").css ({
            top: (_height - boxHeight) / 2 + "px",
            left: (_widht - boxWidth) / 2 + "px"
          });
        };
        //确定按钮事件
        var btnOk = function (callback) {
          $ ("#mb_btn_ok").click (function () {
            $ ("#mb_box,#mb_con").remove ();
            if (typeof callback == "function") {
              callback ();
            }
          });
        };
        //取消按钮事件
        var btnNo = function () {
          $ ("#mb_btn_no,#mb_ico").click (function () {
            $ ("#mb_box,#mb_con").remove ();
          });
        };
      },
      // 批量导入上传
      update() {
        this.toggleLoading ('true');
        var file = document.getElementById ("filebutt").files[0];
        var param = new FormData (); //创建form对象
        param.append ("fileinput", file, file.name); //通过append向form对象添加数据
        param.append ("chunk", "0"); //添加form表单中其他数据
        var config = {
          headers: {
            "Content-Type": "multipart/form-data"
          }
        }; //添加请求头
        axios
          .post (api.domainUrl + "api/Upload/Upload", param, config)
          .then (response => {
            var istype = response.data.type;
            $ ("#myModal3").modal ("hide");
            if (istype == "1") {
              $.MsgBox.Alert ("提示", "导入成功");
            } else if (istype == "2") {
              $.MsgBox.Alert ("2", "模板不符");
            } else {
              var data = response.data.url;
              $.MsgBox.Alert ("提示", "部分数据导入失败，请下载错误信息", data);
            }
            this.toggleLoading ('false');
            this.loaddata (1);
          })
          .catch (err => {
            console.log (err);
          });
      },

      changeform: function (param, value) {
        var query = location.search.substring (1);
        var p = new RegExp ("(^|&" + param + ")=[^&]*");
        if (p.test (query)) {
          query = query.replace (p, "$1=" + value);
          location.search = "?" + query;
        } else {
          if (query == "") {
            location.search = "?" + param + "=" + value;
          } else {
            location.search = "?" + query + "&" + param + "=" + value;
          }
        }
      },
      //入库    二维码下拉框

//      assetqrcodeLoad() {
//        var _self = this;
//        var params = {};
//        api.Assetinfo
//          .GetQrCode (params)
//          .then (res => {
//            if (res.data.flag == 1) {
//              if (res.data.success == 1) {
//                _self.assetqrcodedata = res.data.data.qrcode;
//                //this.addbizsalesdata = result.page.data;
//                //rqcodeinfoid
//                var len = _self.assetqrcodedata.length;
//                var CompanentId = "selFxs";
//                var optionString = "<option value=''>" + "请选择" + "</option>";
//                for (var i = 0; i < len; i++) {
//                  // optionString += "<option value=\'" + this.assetqrcodedata[i].rqcodeinfoid + "\'>" + this.assetqrcodedata[i].rqcodeinfonumber + "</option>";
//                  optionString +=
//                    "<option value='" +
//                    _self.assetqrcodedata[i].rqcodeinfoid +
//                    "'>" +
//                    _self.assetqrcodedata[i].rqcodeinfonumber +
//                    "</option>";
//                }
//                var myobj = document.getElementById (CompanentId);
//                if (myobj.options.length == 0) {
//                  $ ("#" + CompanentId).html (optionString);
//                  $ ("#" + CompanentId).selectpicker ("refresh");
//                }
//                $ ("#" + CompanentId).html (optionString);
//                $ ("#" + CompanentId).selectpicker ("refresh");
//              } else {
//                return false;
//              }
//            }
//          })
//          .catch (error => {
//            console.log (error);
//          });
//      },

      ApiaClick() {
        window.location.href = api.domainUrl + "pages/OrdinaryUsers/Home.html";
      }
    },
    created: function () {
      //郭超改部分
      this.url = api.domainUrl + '/api/AcceptanceForm/SolrList';
      this.categoryInfoLoad ();
      this.AlertBox ();
      this.usertoken = api.GetQueryString('usertoken');
      /*this.assetqrcodeLoad (); //下拉框二维码*/

//      this.assetqrcodeLoad (); //下拉框二维码

    }
  });
  //资产类别树状方法
  var setting = {
    view: {
      selectedMulti: false
      //clickExpand:true,
    },
    data: {
      key: {
        title: "t"
      },
      simpleData: {
        enable: true
      }
    },
    callback: {
      beforeClick: componet.methods.beforeClick
    }
  };
</script>
<style>
  center {
    padding: 5px 0;
  }
  #loadingDiv {
    filter: alpha(opacity=50);
    background: rgba(0, 0, 0, 0);
    position: absolute;
    top: 0px;
    left: 0px;
    width: 100%;
    height: 100%;
    z-index: 100;
  }

  .loading {
    width: 160px;
    height: 56px;
    position: absolute;
    top: 50%;
    left: 50%;
    margin-left: -80px;
    line-height: 56px;
    color: #fff;
    padding-left: 60px;
    font-size: 15px;
    background: #000 url(../assets/loader.gif) no-repeat 20px 50%;
    background-size: 25px 25px;
    opacity: 0.7;
    z-index: 100;
    -moz-border-radius: 20px;
    -webkit-border-radius: 20px;
    border-radius: 20px;
    filter: progid:DXImageTransform.Microsoft.Alpha(opacity=70);
  }
</style>
