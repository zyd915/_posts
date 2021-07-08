---
title: IntellJ IDEA  激活码2021 每日更新 长期提供【JetBrains全家桶】
date: 2020-11-01 00:23:49
updated: 2020-11-01 00:23:50
permalink: code
thumbnail: https://static.studytime.xin//studytime/image/articles/qD3FGq.jpg
tags: 
    - jetbrains
categories: tool
keywords: idea激活码,激活码,phpstorm激活码,pycharm激活码,IntelliJ激活码,golang激活码,全家桶
toc: true
top: 120
excerpt: jetbrains 全家桶激活码，实测可用，每日都会更新，长期提供。再也不用怕突然激活码失效的尴尬了。                      
---
<link rel="stylesheet" href="https://static.studytime.xin/hexo/css/bootstrap.min.css" />


### 打开Jetbrains软件，支持激活码如：PHPstorm激活码、IntelliJ IDEA激活码、Golang激活码、Pycharm激活码、Webstorm激活码等

### 点击activation code

### 点击下下方获取激活码

<!-- 按钮触发模态框 -->
<button class="btn btn-primary btn-lg" data-toggle="modal" data-target="#myModal" style="width: 100%">
	点击获取激活码(有效期至2021年10月)
</button>

<button class="btn btn-primary btn-lg" data-toggle="modal" data-target="#myCodeModal" style="width: 100%;margin-top: 25px;">
	永久激活方式（最新）
</button>

<!-- 模态框（Modal） -->
<div class="modal fade" id="myModal" tabindex="-1" role="dialog" aria-labelledby="myModalLabel" aria-hidden="true">
	<div class="modal-dialog">
		<div class="modal-content">
			<div class="modal-header">
				<h4 class="modal-title" id="myModalLabel">
					激活码
				</h4>
			</div>
			<div class="modal-body">
			    <img src="https://static.studytime.xin//studytime/image/articles/rR0UDK.jpg" style="margin-left:25%;width: 40%;height: 40%;margin-bottom:15px;" />
			    <ol>
			        <li>目前有效期到2021年10月份，本激活码会持续不断更新。</li>
			        <li>失效请留言，作者会进行更新，若等不及更新，可选择获取永久激活方式</li>
			    </ol>
				<input type="password" class="form-control" id="inputPassword" placeholder="请输入密码" style="text-align: center">
				 <figure class="highlight shell code_input" style="display:none">
				 <table>
                 <tr class="jetbrains-code">
                     <td class="gutter"><pre><span class="line">1</span><br></pre></td>
                  </tr>
                  </table>
                 </figure>
				 <td class="code jetbrains-code"></td>
			</div>
			<p style="text-align: center;color: red">提交获取激活码时，会有延迟，请不要关闭窗口</p>
			<div class="modal-footer">
				<button type="button" class="btn btn-default" data-dismiss="modal">关闭
				</button>
				<button type="button" class="btn btn-primary" data-dismiss="submit-modal" onclick="show_active()">
					提交
				</button>
			</div>
		</div><!-- /.modal-content -->
	</div><!-- /.modal -->
</div>


<!-- 模态框（Modal） -->
<div class="modal fade" id="myCodeModal" tabindex="-1" role="dialog" aria-labelledby="myCodeModalLabel" aria-hidden="true">
	<div class="modal-dialog">
		<div class="modal-content">
			<div class="modal-header">
				<h4 class="modal-title" id="myCodeModalLabel">
					永久激活方式
				</h4>
			</div>
			<div class="modal-code-body">
			    <img src="https://static.studytime.xin//studytime/image/articles/gdvrpv.jpg" style="margin-left:25%;width: 40%;height: 40%;margin-bottom:15px;" />
			</div>
			<div class="modal-footer">
				<button type="button" class="btn btn-default" data-dismiss="modal">关闭</button>
			</div>
		</div><!-- /.modal-content -->
	</div><!-- /.modal -->
</div>


<script>
   function show_active() {
           var passwd = $.trim($('#inputPassword').val());
           if(!passwd)
           {
               alert('密码不能为空！');
               return false;
           }
            $.ajax({
                     async : true,
                     type:"get",
                     url:"https://api.studytime.xin/activationCode?passwd=" + passwd,
                     data:{},
                     dataType:"json",
                     success:function (e) {
                         if (e.status == 0) {
                            alert( e.message);
                            return false;
                         }else {
                            $('.jetbrains-code').append( "<td class=code><pre><span class=line>" +e.data+"</span><br></pre></td>");
                            $('.code_input').show();
                         }
                     },
                     error:function (e) {
                        alert( e.message);
                        return false;
                     }
            });
   }
</script>




